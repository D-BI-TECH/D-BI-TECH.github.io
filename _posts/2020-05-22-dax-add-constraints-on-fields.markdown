---
layout: post
title:  DAX进阶：为字段添加约束
date:   2020-05-22 03:03:50 +0000
image:  09.jpg
tags:   [DAX,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

（本文以解决在诸如高并发等场景下造成的负库存问题为例，讲解用DAX为新建字段设置检查约束的方法）

### 前述

年前，有人在知识星球问过我如何用DAX计算累计库存变动，并且避免负库存，当时解决问题后并没有专门为此写技术分享，但近期我又收到了类似的提问，因此觉得有必要写一篇博客，讲解此情况下如何用DAX为计算列设置非负约束。

### 面临的问题

在企业环境中， 有许多场景需要BI人员为表中的字段添加检查约束， 比如在零售行业的促销活动中，就可能会遇到高并发场景，因超卖而产生的负库存问题，这时，必须要提前为库存值设置检查约束，当某产品库存小于0时，强制将其设置为0，避免库存为负；有些企业可能会为某些产品设置安全库存，这时约束条件可能是大于3，大于5或其他值等等，当然，还有很多需要为其他的为字段设置约束的场景。通常情况下，这种检查约束都会在数据仓库中得到处理,  比如在SQL Server， 只需运行一条CHECK语句，就可以为表中指定字段添加约束条件， 甚至在Excel中，也是简单到一行公式就可以完成。如下图所示，2020年1月2日时A产品的库存为4，第二天库存减去5件，出现负库存，而此时只需要写一条IF公式向下填充，负库存的问题就可以解决：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020052016515058.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

然而在Power BI或SSAS表格模型中，DAX引擎的本质使得这种计算实现起来十分困难，比如前文所述的例子，在1月3日，你要如何根据1月2日的剩余库存，加上当天的库存净变化，得出1月3日的剩余库存，而1月4日的库存又要依据1月3日的库存结果来计算，以此类推，到最后一行，1月10日的剩余库存需要依据1月9日的剩余库存加上当日的库存净变化，所以对于库存列而言，其计算是一个不断依赖自身逐行迭代的结果。

幸运的是，该问题早已得到解决，我在过去的文章[《解决DAX中数据的循环迭代问题》](https://d-bi.gitee.io/dax-iteration-calculate/)提到过：“迭代的本质是数列”，依照这个思路，该问题得到解决，然而在本案例中，在迭代的基础上，还需要给该列添加非负约束，这使得问题变得更加复杂，不仅因为在Power BI数据模型之中, 你不可能像Excel那样可以将公式简单地向下填充，而且过去的数列式思维也很难得到应用，可以说该问题充分体现了DAX"简洁而笨重"的特性。那么，真的没有解决办法了吗？

### 数学问题 数学解决

首先，在Power BI直接模拟一个库存进出明细表：

```SQL
Stock_NetChange = 
DATATABLE(
"Item",STRING,
"Date",DATETIME,
"InitialStock",INTEGER,
"NetChange",INTEGER,
{
    {"A","2020-01-01",1,1}
   ,{"A","2020-01-02",1,2}
   ,{"A","2020-01-03",1,-5}
   ,{"A","2020-01-04",1,3}
   ,{"A","2020-01-05",1,-5}
   ,{"A","2020-01-06",1,7}
   ,{"A","2020-01-07",1,2}
   ,{"A","2020-01-08",1,-10}
   ,{"A","2020-01-09",1,3}
   ,{"A","2020-01-10",1,6}
   ,{"B","2020-01-01",2,1}
   ,{"B","2020-01-02",2,-4}
   ,{"B","2020-01-04",2,7}
   ,{"B","2020-01-07",2,-1}
   ,{"B","2020-01-09",2,-4}
})
```

如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520184848269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

首先，新建一列用以计算每种产品的每日剩余库存，此时不考虑库存的非负约束问题：

```SQL
StockCumSum = 
VAR _Initial =
    FIRSTNONBLANK ( Stock_Demo[InitialStock], 0 )
VAR _Sum =
    CALCULATE (
        SUM ( Stock_Demo[NetChange] ),
        FILTER (
            ALL ( Stock_Demo ),
            AND (
                Stock_Demo[Date] <= EARLIER ( Stock_NetChange[Date] ),
                Stock_Demo[Item] = EARLIER ( Stock_NetChange[Item] )
            )
        )
    )
RETURN
    _Initial + _Sum
```

如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520190434529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

可以发现，在没有约束条件下，剩余库存按照正常的计算方法累积得出，因此在1月3号那一行就得出剩余库存为-1的结果值，而我们的想要的结果是，因为4加负5得负1，出现负库存因此需要改为0，此外，下一日的计算也要依据0而非负1，因此，从出现负数之日起，后面的每个结果都是错误的，怎么解决呢？**此时，我们需要改变一下思维，不能单纯只考虑DAX的技术实现，也要结合一点数学思维**。思考一下，StockCumSum和正确的计算结果的差距是如何形成的，答案是：“不断累积的负值”， 那么如果我们把这个累积的差距算出来，再和原来的StockCumSum相加，不就得出正确的结果了吗？按照这个思路，新建列如下：

```SQL
Result_Diff = 
IF(Stock_NetChange[StockCumSum]<0,
	ABS(Stock_NetChange[StockCumSum]),0)
```

结果如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200520192248111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

这一列，就是因负库存导致的和正确结果的净值差距。但我们知道，每日的剩余库存是根据每日库存净变化量，也即NetChange累积计算的，因此差距也是累积的，另一点需要注意的是（你也可以通过数学公式的推导），在没有非负约束条件下的库存结果（StockCumSum）和有非负约束的库存结果的差距，取决于当日以前的最大净值差距。什么意思呢？比如1月7日这一天，StockCumSum的值为6，Result_Diff在此之前的最大值为3，因此3就是StockCumSum和非负约束的库存结果的差距，也就是说，按照非负约束原则，1月7日的剩余库存应该为9. 同理，其他日期的计算也是相同原则，于是可以建立如下公式：

```SQL
Result_Diff_Max = 
VAR _MAX =
    CALCULATE (
        MAX ( Stock_NetChange[Result_Diff] ),
        FILTER (
            ALL ( Stock_NetChange ),
            AND (
                Stock_NetChange[Date] <= EARLIER ( Stock_NetChange[Date] ),
                Stock_NetChange[Item] = EARLIER ( Stock_NetChange[Item] )
            )
        )
    )
RETURN
    _MAX
```

结果如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521110906609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

现在我们把StockCumSum和差距值相加，就可以得出在避免负库存情况下的剩余库存值了，于是上一条公式改为：

```SQL
StockResult = 
VAR _MAX =
    CALCULATE (
        MAX ( Stock_NetChange[Result_Diff] ),
        FILTER (
            ALL ( Stock_NetChange ),
            AND (
                Stock_NetChange[Date] <= EARLIER ( Stock_NetChange[Date] ),
                Stock_NetChange[Item] = EARLIER ( Stock_NetChange[Item] )
            )
        )
    )
RETURN
    _MAX + Stock_NetChange[StockCumSum]
```

最终结果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200521111615558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

手动验证一下，计算无误，看似复杂的DAX问题结合一点数学思维就得到了完美的解决。另外，本案例的约束条件是值必须大于等于0，你完全可以根据你的需求，按照本案例的原理改成其他约束条件，比如你可以要求所有产品设定5件的安全库存，只需要在上文公式中必要处把0改成5，再稍微改下Result_Diff公式即可，如果你需要为不同的产品设置不同的值约束也是没有问题的，只需要引用对应的约束值字段即可。

### 总结

在企业环境中，当然还是尽量将这种约束处理交给数据仓库处理，但正如我先后收到的两次类似的问题，还是有不少用户不具备这种条件，因此不得不把某些复杂的数据操作交给前端的DAX进行处理，不过事实上本文不仅在于解决了负库存甚至DAX处理字段约束问题，更重要的是，在解决复杂计算问题是，引入数学思维是十分必要的, "为何在Excel如此简单的操作放到DAX就那么难？", 当你下次再发出这样的感慨时，回顾本文，理清思路，相信问题一定能够迎刃而解！

*本文首发于【D-BI】(本站)与【PowerBI星球】微信公众号*
