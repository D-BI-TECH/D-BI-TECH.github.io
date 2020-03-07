---
layout: post
title:  DAX计算重复购买率与新老客自动分类
date:   2019-03-24 12:03:50 +0300
image:  07.jpg
tags:   [Power BI,Power Pivot,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

## 简述
重复购买率（*二次购买率*）及新老客户占比都是客户数据分析中极其重要的指标（*除此之外没什么好简述的，直接上货*）
## 目标
1.把订单分为客户首次购买的订单和后续购买的订单，进而算出重复购买率
2.把客户分为新客户与老客户，进而计算新老客占比

## 过程
还是原来的数据源：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190324193737229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

对于判断哪些订单是用户首次购买的，哪些不是首次购买的，思路是分别为客户ID和订单日期创建变量，变量可以保存在当前筛选上下文的计算列，Filter可以使计算处于新的筛选上下文，对于客户ID，让它的变量（*旧的上下文*）和它在新的上下文进行匹配，对于订单日期，让它的变量大于新上下文的订单日期（*同样的原理，不使用变量而用earlier函数代替也可以，但本人更推荐使用变量*）在这个基础上使用SUMX迭代，判断结果的行数是否大于零，如果不大于零，说明对于某一客户没有任何订单是在与首单不同的日期生成的，反之同理：

>```Python
>二次购买判断 = 
>VAR
>E_Date = 'Data'[订单日期]
>VAR
>CUST = 'Data'[客户 ID]
>RETURN
>IF(
>    SUMX(
>        FILTER('Data',CUST = 'Data'[客户 ID]&&E_Date > 'Data'[订单日期]),
>        COUNTROWS('Data'))>0,"非首次","首次")
```

效果如下：
（*这里需要补充本案例的一个逻辑，所谓非首次购买是指和首单不同日期的订单，比如新客户当天下了两单，第二单由于和第一单是同一天，因此这一单算入首次，而非第二次*）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127173026217.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

接下来就可以计算重复购买率了：  

>```Python
>重复购买率 = 
>DIVIDE(
>    CALCULATE(
>        DISTINCTCOUNT(Data[客户 ID]),'Data'[二次购买判断] = "非首次"),
>    DISTINCTCOUNT(Data[客户 ID])
>)
>```

效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127172450397.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

- [x] 把订单分为客户首次购买的订单和后续购买的订单，进而算出重复购买率
- [ ] 把客户分为新客户与老客户，进而计算新老客占比

新老客的划分和首单的划分在逻辑上有些不同，对于同一个客户，如果他第一天下了一单，那么无论该客户在此后的日子里下了多少单，此单都会永远会被划为首单，而他的客户身份则不同，当他变为老客户时，原来被标记为新客户的那个首单就要变更为老客户了。这是因为，当我们想要分析老客户的购买记录时，我们希望看到的是他完整的下单记录。公式原理方面则类似：

>```Python
>新老客判断 = 
>if(
>    calculate(
>        distinctcount('Data'[订单日期]),
>            filter('Data','Data'[客户 ID]=earlier('Data'[客户 ID])))>1,
>            "老客户","新客户")
>```

结果如下表：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127173535573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

最后，分别计算出新老客户的占比即可：

>```Python
>老客户占比 = 
>DIVIDE(
>    CALCULATE(
>        DISTINCTCOUNT(Data[客户 ID]),'Data'[新老客判断] = "老客户"),
>    DISTINCTCOUNT(Data[客户 ID]))
>```

>```Python
>新客户占比 = 
>DIVIDE(
>    CALCULATE(
>        DISTINCTCOUNT(Data[客户 ID]),
>        'Data'[新老客判断] = "新客户"),
>    DISTINCTCOUNT(Data[客户 ID]))
>```

- [x] 把订单分为客户首次购买的订单和后续购买的订单，进而算出重复购买率
- [x] 把客户分为新客户与老客户，进而计算新老客占比

## 其他
对于新老客户分类这一块，也许不同的地方有不同的计算逻辑，但基本原理都是一样的。如果大家有更好的方法，希望能得到分享:)