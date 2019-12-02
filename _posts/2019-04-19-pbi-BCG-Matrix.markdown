---
layout: post
title:  DAX波士顿矩阵可视化实现
date:   2019-04-19 06:03:50 +0000
image:  11.jpg
tags:   [Power BI,DAX,可视化]
---

<small>*关于本文内容，本是此前有人提出过类似问题，后来自己想到了一种解决办法，觉得可以拿出来分享一下，因此就写了这篇短文。希望能够帮到有需要的人。*<small>

波士顿矩阵示意图(BCG Matrix)：

![图片源自http://ideas.sdabocconi.it](https://img-blog.csdnimg.cn/20191129160615223.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

波士顿矩阵(四象限分析法)是非常经典的市场管理策略，通过把产品分成明星，问题，现金牛和瘦狗四种类型，以利于企业针对不同产品制定有效的策略。
矩阵的横轴通常为市场份额，纵轴为市场增长率，用DAX实现如下：

>```Python
--市场份额
Sales % = 
DIVIDE(
    sum(Orders[Quantity]),
    CALCULATE(sum(Orders[Quantity]),
        ALLEXCEPT(Orders,
            'Date'[Month]))) -- 因市场增长率按上月环比计算，故此度量值需要能够被月份筛选
>```
--------------------------------------------------------------------------------------------------
>```Python
--市场增长率
SalesGrowth % = 
DIVIDE(
        (SUM(Orders[Quantity])-
            CALCULATE(SUM(Orders[Quantity]),
                DATEADD('Date'[Date],-1,MONTH))),
        CALCULATE(sum(Orders[Quantity]),
            DATEADD('Date'[Date],-1,month))
    )
>```

运行后可得出下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129160651563.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

每一个散点代表一个具体产品，实际应用中可以设置上钻到品类
现在的问题是，需要以市场份额(Sales %)的平均值以及市场增长率(SalesGrowth %)的平均值为界限，把散点分成波士顿矩阵的四大类，每个类别用不同颜色表示。在PowerBI，按照度量值设置图形颜色可以先把图形转成柱状图，这样就可以在"Data colors"处看到一个条件格式的选项：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129160706614.png)

比如下图，选择"Qty"作为评判数据的颜色格式的标准，当它大于10时，数据显示湖绿色。这样，柱图就会按照颜色分类，然后再把数据转回散点图，就实现了按我们设定的度量值公式决定散点颜色的效果了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129160716921.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

然而，真正关键的问题出现了。截至2019年4月，该选项框中只有一个可以供我们选择字段或度量值的选项框，但我们需要的是，数据要依据两个度量值分类，即市场份额(Sales %)的平均值以及市场增长率(SalesGrowth %)的平均值，在这种情况下，这种方法似乎根本行不通。但其实，让一个度量值实现两个度量值的效果是可以做到的。
设想一下，波士顿矩阵的形态是二维的,如下示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129160732216.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

而其实，它可以转换成一维的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112916163422.png)

因此，我们可以在公式中给每个产品分成ABCD四类，分别代表矩阵中的每一个大类，然后，计算出每个散点与原点的距离，依据不同的大类，给予不同的分离值，A最小，D最大，如上图一字排开，就可以成功把矩阵转化为可以由一个度量值决定颜色格式的一维形态。此外，需要给四大类数据分别使用对数函数，这样可以避免原本属于A类的产品由于数据较大而被错误分类到B,C甚至D的情况。
依照这一理论，编写公式如下：

>```Python
BCG_COLOR = 
var vt = SUMMARIZE('Date','Date'[Month])
var GrowthAvg = 
CALCULATE(
AVERAGEX(vt,
    'Orders'[SalesGrowth %]
    ),
all(Orders))
var SalesAvg = 
CALCULATE(
    AVERAGEX(Orders,
        CALCULATE([Sales %],
            ALLEXCEPT(Orders,Orders[Product Name]))),
    ALLEXCEPT(Orders,'Date'[Month]))
var distance = ([SalesGrowth %]^2 + [Sales %]^2) ^ 0.5 + 1
var result = 
if(and([SalesGrowth %]>=GrowthAvg,[Sales %]>=SalesAvg),"A",
if(and([SalesGrowth %]>=GrowthAvg,[Sales %]<SalesAvg),"B",
if(and([SalesGrowth %]<GrowthAvg,[Sales %]>=SalesAvg),"C","D")
)
)
return
if(distance = 0,BLANK(),
switch(result,
    "A",LOG10(distance),
    "B",LOG10(distance)+100,
    "C",LOG10(distance)+200,
    log10(distance)+300))
>```

运行代码，把该度量值设置在颜色格式上：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129161814262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

效果如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129161919697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)