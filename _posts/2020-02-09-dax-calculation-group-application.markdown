---
layout: post
title:  DAX：Calculation Group 概念与应用
date:   2020-02-09 04:03:50 +0000
image:  11.jpg
tags:   [DAX,SSAS]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

*【前述：根据微软Power BI排期,原本只能在SSAS 2019中使用的DAX特性--CalulationGroup,将在今年的Desktop中发布。本文将围绕CalulationGroup,言简意赅的讲解它的概念,以及它在PBI报表中具体是如何被应用的,能为我们带来什么。】*

众所周知,近期国内疫情严重,宅在家里,工作要做,游戏要玩,但对BI技术的探索也不能少。偶然想起半年前Macro Russo大师曾发表数篇关于CalulationGroup的介绍性文章,但当时本人除了Power BI以外,SSAS也只装了2012和2017两个版本,不便亲自实践,只能在理论意义上理解CalulationGroup,现在有空折腾了,就在我的个人电脑里又装了一个SQL Server 2019,以SSAS表格模型作为后端,Power BI作为前端来实际体验一下CalulationGroup。

### 概念

关于CalulationGroup,中文译为“计算组",也称”度量值组",在Macro的[此篇文章](https://www.sqlbi.com/articles/introducing-calculation-groups/)中有很好的介绍,推荐阅读,本文在概念上仅简明介绍关于它的几个核心知识点：

- CalulationGroup是一个表,这意味着它同样可以在DAX公式中被引用；它包含一列或两列,第一列即实际意义上的"度量值组",列中的值由多个度量值组成；第二列为可选列,用于对度量值组进行排序。
- 度量值组中的值,即度量值本身被成为CalulationItem(计算项),它本身代表的是一个上下文环境。[SELECTEDMEASURE()](https://docs.microsoft.com/zh-cn/dax/selectedmeasure-function-dax)函数接受度量值作为参数,在特定计算项的上下文环境中执行计算。
- 计算项默认对所有度量值有效,但你可以使用[ISSELECTEDMEASURE()](https://docs.microsoft.com/zh-cn/dax/isselectedmeasure-function-dax)或[SELECTEDMEASURENAME()](https://docs.microsoft.com/zh-cn/dax/selectedmeasurename-function-dax)对计算项的应用范围做出限制,使之仅对特定度量值有效。
- 允许创建多个度量值组,但必须设定优先级,以使引擎能够理解特定度量值在不同度量值组之下的计算项上下文环境中的计算顺序（如X+1和X * 2, 是先加1还是先乘2,需要设置清楚）。在SSAS中,优先级参数为Precedence, 该值越大,优先级越高。
- 同一个度量值组中,只能有一个计算项被激活,这是可以理解的,因为在同一个度量值组中,不同计算项不存在优先级关系,一旦多个计算项被同时应用,引擎无法理解哪个计算项被优先应用。

此外,还有一个名为[SELECTEDMEASUREFORMATSTRING()](https://docs.microsoft.com/zh-cn/dax/selectedmeasureformatstring-function-dax)的函数,它代表当前度量值的格式字符串,我们暂时无法了解未来CalulationGroup在Power BI中是如何使用的,但在SSAS中,你可以为每个计算项使用DAX表达式来设定其格式字符串。

### 应用
 
首先介绍本文使用微软官方提供的AdventureWork2017作为数据源进行建模,主要表格关系如下示：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200208205319212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

然后,在应用场景之下,你可能会想,CalulationGroup最主要的作用是什么？它大幅减轻了DAX代码维护成本,在越复杂的报表项目中,可能涉及到要建立许多个度量值时,这种作用的优势越明显,此外,它还增强了对数据格式控制的灵活性。在下文中,我将使用CalulationGroup为Power BI报表实现三种效果,这些效果不仅是在应用中经常需要实现的,还可以很好的表现其优势：

 - 建立多个度量值,其中包括销量,销售额,税后销售额,成本及利润,并为每个度量值计算其上月值,上月环比,去年当月值,去年同比以及度量值本身的值。
 - 实现数据可以依据不同的日期字段进行切换：根据订单日期,根据发货日期以及根据截止日期
 - 实现汇率转换,同时需要依据不同的货币切换货币符号,以人民币,美元及欧元为例
 
最终效果的静态展示图如下：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209000825729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)
 
##### 1. 建立多个度量值,其中包括销量,销售额,税后销售额,成本及利润,并为每个度量值计算其上月值,上月环比,去年当月值,去年同比以及度量值本身的值。

通常情况下,实现此效果需要建立(5*5)共25个度量值！或者需要利用计算表及SWITCH()编写较为冗长的代码,十分不便,利用CalulationGroup,你只需要设定销量,销售额,税后销售额,成本及利润这五个基础度量值就足够了,然后创建一个CalulationGroup,分别代表不同的计算方式,即可实现此效果。在本例中,此CalulationGroup的表达式为：
 

>```Python
CalulationGroup 1：
--共有五个计算项
This Month:=
IF(ISBLANK(SELECTEDMEASURE ()),0,SELECTEDMEASURE ())
---------------------------------------------------------------
Last Month:=
VAR LAST_MONTH = 
CALCULATE (
        SELECTEDMEASURE (), 
        DATEADD('DimDate'[FullDateAlternateKey],-1,MONTH)),
RETURN
IF(ISBLANK(LAST_MONTH),0,LAST_MONTH)
---------------------------------------------------------------
MOM:=
VAR THIS_MONTH = SELECTEDMEASURE(),
VAR LAST_MONTH =
    CALCULATE (
        SELECTEDMEASURE (),
        DATEADD('DimDate'[FullDateAlternateKey],-1,MONTH)
    )
RETURN 
IF(
        ISBLANK(LAST_MONTH),
        BLANK(),DIVIDE(THIS_MONTH,LAST_MONTH)-1
    )
---------------------------------------------------------------
SPLY:=
VAR LAST_YEAR = 
CALCULATE (
        SELECTEDMEASURE (), 
        DATEADD('DimDate'[FullDateAlternateKey],-1,YEAR))
RETURN
IF(ISBLANK(LAST_YEAR),0,LAST_YEAR)
---------------------------------------------------------------
YOY:=
VAR THIS_YEAR = SELECTEDMEASURE()
VAR LAST_YEAR =
    CALCULATE (
        SELECTEDMEASURE (), 
        DATEADD('DimDate'[FullDateAlternateKey],-1,YEAR)
    )
RETURN 
IF(
        ISBLANK(LAST_YEAR),BLANK(),
        DIVIDE(THIS_YEAR,LAST_YEAR)-1
    )
>```

所有基础度量值会在以上任一被选中的计算项中执行计算,比如选择SPLY(上年同月)计算销量,则执行以下计算：

>```Python
SPLY:=
VAR LAST_YEAR = 
CALCULATE (
        FactInternetSales[Sales Qty], 
        DATEADD('DimDate'[FullDateAlternateKey],-1,YEAR))
RETURN
IF(ISBLANK(LAST_YEAR),0,LAST_YEAR)
>```

在Power BI前端,选择任意月份,效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020822441210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这里可以发现一个问题,即计算项并未按照我在度量值组中所设定的顺序显示,这时就需要设定CalulationGroup中第二列的值,即ordinal参数,该参数的设定在使用Visual Studio 2019开发环境中的SSAS中不可见,因此需要在模型部署的源代码中补上这一参数(相信未来在Power BI中关于CalulationGroup的设定能够更加友好),设置后效果如下：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200208230156324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

##### 2.实现数据可以依据不同的日期字段进行切换：根据订单日期,根据发货日期以及根据截止日期

实现此需求,自然需要用到[USERALATIONSHIP()](https://docs.microsoft.com/zh-cn/dax/userelationship-function-dax),同理,实现此效果需要新建一个CalulationGroup,注意,此时以及存在超过一个CalulationGroup了,因此必须指定好它的优先级。表达式如下：

>```Python
CalulationGroup 2:
BY OrderDate:=
SELECTEDMEASURE()
---------------------------------------------------------------
BY ShipDate:=
CALCULATE(
    SELECTEDMEASURE(),
    USERELATIONSHIP(FactInternetSales[ShipDate],
    DimDate[FullDateAlternateKey]))
---------------------------------------------------------------
BY DueDate:=
CALCULATE(
    SELECTEDMEASURE(),
    USERELATIONSHIP(FactInternetSales[DueDate],
    DimDate[FullDateAlternateKey]))
>```

##### 3.实现汇率转换,同时需要依据不同的货币切换货币符号,以人民币,美元及欧元为例

此处的难点首先在于转换汇率时,度量值组如何忽略对非金额数据的影响,如本例中的Sales Qty,这一点可以使用前文提到的ISSELECTEDMEASURE()解决,DAX表达式如下：

>```Python
CalulationGroup 3:
--注：数据源中的汇率表以美元作为本位币
RMB :=
IF (
    NOT ISSELECTEDMEASURE ( [Sales Qty] ),
    DIVIDE (
        SELECTEDMEASURE () * [Avg_CurrencyRate],
        CALCULATE (
            MAX ( FactCurrencyRate[AverageRate] ),
            FILTER ( 'FactCurrencyRate', 'FactCurrencyRate'[CurrencyKey] = 103 )
        )
    ),
    SELECTEDMEASURE ()
)
---------------------------------------------------------------
EURO:=
IF (
    NOT ISSELECTEDMEASURE ( [Sales Qty] ),
    DIVIDE (
        SELECTEDMEASURE () * [Avg_CurrencyRate],
        CALCULATE (
            MAX ( FactCurrencyRate[AverageRate] ),
            FILTER ( 'FactCurrencyRate', 'FactCurrencyRate'[CurrencyKey] = 36 )
        )
    ),
    SELECTEDMEASURE ()
)
---------------------------------------------------------------
USD:=
IF(
    NOT ISSELECTEDMEASURE([Sales Qty]),
    SELECTEDMEASURE()*[Avg_CurrencyRate],SELECTEDMEASURE()
    )
>```

其次在CalulationGroup 1中,MOM和YOY是百分比值,由于度量值组中所设定的格式字符串会覆盖度量值本身的格式,因此设定货币符号的格式字符串同时也会影响到MOM和YOY,因此必须要找到办法能够判断哪些计算项需要货币符号,哪些不需要。事实上,如前文所述,CalulationGroup是一个表,因此结合[SELETEDVALUE()](https://docs.microsoft.com/zh-cn/dax/selectedvalue-function)函数即可解决问题,对应的格式字符串表达式如下：

>```Python
CalulationGroup 3 (FormatString表达式):
--注：报表所有金额基础度量值默认格式为货币（人民币）
RMB:=
SELECTEDMEASUREFORMATSTRING()
---------------------------------------------------------------
EURO:=
IF(ISSELECTEDMEASURE([Sales Qty]),
    SELECTEDMEASUREFORMATSTRING(),
    IF(
        NOT OR(SELECTEDVALUE('CalculationGroup 1'[CalculationItemColumn 1]) = "MOM",
            SELECTEDVALUE('CalculationGroup 1'[CalculationItemColumn 1]) = "YOY"),
            "€#,0.00",
            SELECTEDMEASUREFORMATSTRING()
        )
    )
---------------------------------------------------------------
USD:=
IF(ISSELECTEDMEASURE([Sales Qty]),
    SELECTEDMEASUREFORMATSTRING(),
    IF(
        NOT OR(SELECTEDVALUE('CalculationGroup 1'[CalculationItemColumn 1]) = "MOM",
            SELECTEDVALUE('CalculationGroup 1'[CalculationItemColumn 1]) = "YOY"),
            "$#,0.00",
            SELECTEDMEASUREFORMATSTRING()
        )
    )
>```

至此,我们即可实现所有效果,但事情到此尚未结束,在下图中你会发现在计算销售额的MOM和YOY时出现计算错误：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200208235649720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

之前所设定的公式都是没有问题的,但为何会出现错误呢？原因是在计算环比和同比时,所基于的数据是本币数据,而非转换为人民币,美元或者欧元后的数值,因此罪魁祸首是对CalulationGroup的优先级的错误设定。我们需要让数据先转换为指定货币,然后再执行计算。因此这里需要将CalulationGroup 3的优先级设为最低,CalulationGroup 2其次。如下图所示,数据得到完美的纠正：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200209000705808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

报表最终如下：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020020900451420.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 总结

利用CalulationGroup,原本需要建立上百个度量值(5 * 5 * 3 * 3)的需求,成功简化为几个基础度量值和三个度量值组,大幅节约了报表开发的时间成本以及后期代码的维护成本；同时,利用格式字符串表达式可以实现报表更灵活更定制化的显示效果。此外,如果你有关于CalulationGroup其他的典型应用案例欢迎留言分享。

*本文在【D-BI】(本站)与[【PowerBI星球公众号】](https://mp.weixin.qq.com/s?__biz=MzA5Mjg0ODA1MA==&mid=2650766974&idx=1&sn=94a16d21b3b363614bb944915aad9ca2&chksm=886d34a3bf1abdb50295ac79dc5da5c2761fbaec30b048ffc0643f05d284098fb1f8f72a9bb4&scene=126&sessionid=1581080153&key=d7c94d8dea45777c40d9084dca859acfde62df616d85a4f32b576f813d12dca639a1e7a3abf31fc1801742672e6995a6b60e3090018d7ca116a0079778b6245295546e2c2f992d2e66a99e1a3532c14a&ascene=1&uin=MjMwMjc5NDY0NA%3D%3D&devicetype=Windows+10&version=6208006f&lang=zh_CN&exportkey=A82e3%2FlCaYPil7FLzq9eduM%3D&pass_ticket=57GzgXzx03QXohqIxp%2F%2FpRb50YEMeUYmZbOYeNA9OPFYRuZedyo1m4mX3S4S5KnD&winzoom=1)同时发布*
