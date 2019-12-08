---
layout: post
title:  用DAX实现TRIMMEAN
date:   2019-09-19 06:03:50 +0000
image:  08.jpg
tags:   [Power BI,SSAS,DAX]
author-name: Daniil Maslyuk
author-image: Daniil.jpg
---

DAX中有多种求均值的方法。一些最流行的方法是平均值（[AVERAGE](https://dax.guide/average/)）、中值（[median](https://dax.guide/median/)）和 mode（没有内置函数；请参见[DAX Patterns](https://www.daxpatterns.com/statistical-patterns/#mode)的示例）。另一个是TRIMMEAN，它存在于Excel中。DAX中没有相应的函数，这篇文章展示了如何在DAX中实现该函数的功能。

微软MVP和BI发烧友Miguel Escobar给我分享了他在[Spanish Power BI论坛](https://foro.poweredsolutions.co/topic/223-representar-la-media-de-los-valores-que-quedan-por-debajo-del-percentil-k/)上的一个帖子，在那里，señor OD_GarcíaDeLaCruz要求我就如何在DAX中从Excel模仿TRIMMEAN函数提出建议。

如果您不熟悉这个函数，我强烈建议您阅读[文档](https://support.office.com/en-us/article/TRIMMEAN-function-D90C9878-A119-4746-88FA-63D988F511D3)。简而言之，该函数计算平均值，同时从尾部排除用户指定的值百分比。

*（译注：此处的"尾部"指数据集的上下两端）*

#### 现存的解决方案

我首先查阅的是DAX Patterns，那里通常是能够提供现成解决方案的很好的来源。不幸的是，并没有关于TRIMMEAN的内容，所以我通过谷歌了解到该问题迄今为止并未充分得到解决。

来自Greg Baldini的解决方案需要您预先知道要排除的值，但如果您希望公式能够处理包含重复值的数据，则可能会出现问题。 另一名微软MVP--Imke Feldmann，她的解决方案在多数场景下都没有问题，但无法处理位于边界的重复值。这是因为在DAX中，没有用户可以访问的行号的概念，所以这个问题需要不同的方法。

*（译注：实现TRIMMEAN的另一种方法是使用TOPN嵌套TOPN，但正如作者所说，这依然无法处理位于边界的重复值，这在具有重复值的数据集中影响了计算精度）*

#### 我的方法

为了使用以下解决方案，需要根据下面的注释替换为你使用的字段：

*（译注：此公式使用了ERROR函数，迄今为止，该函数适用于Power BI及SSAS 2017及以上版本，但在Power Pivot中暂不受支持）*

>```Python
TrimMean = 
VAR TrimPercent = [Percent Value]
-- [Percent Value]：百分比值，介于0到1之间。
VAR Counts =
    SELECTCOLUMNS (
        VALUES ( 'Table'[Column] ),
        "Data Point", 'Table'[Column],
        "Count", CALCULATE ( COUNTROWS ( 'Table' ) )
    )
    --'Table'[Column]：你所引用的字段
    --'Table'：你所引用字段的所属表
VAR NumberOfDataPoints =
    SUMX ( Counts, [Count] )
VAR StartAt =
    INT ( NumberOfDataPoints * TrimPercent / 2 )
VAR FinishAt = NumberOfDataPoints - StartAt
VAR RunningCounts =
    ADDCOLUMNS (
        Counts,
        "RunningCount",
        VAR ThisDataPoint = [Data Point]
        RETURN
            SUMX ( FILTER ( Counts, [Data Point] <= ThisDataPoint ), [Count] )
    )
VAR TrimmedCounts =
    ADDCOLUMNS (
        RunningCounts,
        "Trimmed Count",
        VAR ThisDataPoint = [Data Point]
        VAR MinRunningCount =
            MINX (
                FILTER ( RunningCounts, [RunningCount] >= StartAt ),
                [RunningCount]
            )
        VAR MaxRunningCount =
            MAXX (
                FILTER ( RunningCounts, [RunningCount] <= FinishAt ),
                [RunningCount]
            )
        VAR TrimmedTop =
            MAX ( [RunningCount] - StartAt, 0 )
        VAR TrimmedBottom =
            MAX ( [Count] - MAX ( [RunningCount] - FinishAt, 0 ), 0 )
        RETURN
            SWITCH (
                TRUE,
                [RunningCount] <= MinRunningCount, TrimmedTop,
                [RunningCount] > MaxRunningCount, TrimmedBottom,
                [Count]
            )
    )
VAR Numerator =
    SUMX ( TrimmedCounts, [Data Point] * [Trimmed Count] )
VAR Denominator =
    SUMX ( TrimmedCounts, [Trimmed Count] )
VAR TrimmedMean =
    DIVIDE ( Numerator, Denominator )
VAR Result =
    IF (
        OR ( TrimPercent < 0, TrimPercent >= 1 ),
        ERROR ( "Trim percent must be greater or equal to 0 and less than 1" ),
        TrimmedMean
    )
RETURN
    Result
>```

*（译注：此公式方法巧妙，大于或小于百分比边界的值的TrimmedCounts强制设为0，因此Numerator在求和时通过乘以TrimmedCounts忽视了所有边界以外数据的求和值，然后将剩余求和值Numerator除以剩余值总项数Denominator得出均值。）*

这个公式相当复杂，但希望它很容易使用。上面的公式只适用于物理列。如果要使用类似TRIMMEANX的逻辑，那么Counts变量可能如下所示：

>```Python
VAR Counts =
    GROUPBY (
        SELECTCOLUMNS (
            Your category table expression,
            "Data Point", Expression to average
        ),
        [Data Point],
        "Count", SUMX ( CURRENTGROUP(), 1 )
    )
>```

在数据量较大的表中，性能可能不会太好。如果你有性能更优的方法，告诉我吧！

----------------------
由Davis ZHANG翻译（及注解），[点此](https://xxlbi.com/blog/trimmean-dax/)查看英文原文

----------------------