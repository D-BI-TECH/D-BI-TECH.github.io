---
layout: post
title:  Power BI非常规时间智能场景解决方案：以去年同期最近星期数为例
date:   2020-09-11 01:03:50 +0000
image:  11.jpg
tags:   [Power BI,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*本文主要讲述在一种特殊时间要求下，如何解决数值的计算问题*

### 需求

此处用一个简单的例子来说明本文要解决的问题。假设我们有2019年和2020年所有零售门店的销售数据，我们现在已经拥有了一份PBI报表，上面展示了所有门店在所选时间段的销售额，并且我们使用[SAMEPERIODLASTYEAR](https://docs.microsoft.com/en-us/dax/sameperiodlastyear-function-dax)函数计算了去年同期的销售额，但现在我们新的需求不止于此，我们需要实现当用户选择某一时间段时，比如2020年1月1日到1月5日，要求得出去年此时间段的最近的相同星期数的时间段下的销售额。

是的，这很拗口，但其实不难理解，也就是说我们需要先找到起始日期2020年1月1日，对应到去年是2019年1月1日，由于2020年1月1日是星期三，那么我们需要对应到离2019年2月1日最近的星期三，也就是2019年1月2日，终止日同理，即2019年1月6日，也就是说，如果用户在报表前端选择了2020年1月1日到1月5日，那么需要我们计算出2019年1月2日到2019年1月6日的销售额，这就是本文面临的特殊时间智能计算场景：去年同期最近星期数，它相比普通的SAMEPERIODLASTYEAR计算而言，考虑到了星期的因素，在某些业务场景下更加合理。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911120256898.png#pic_center)


### 方案

根据需求，有两种解决方案，第一种是在后端使用SQL构建一张表，用于展示不同年份日期在星期数上的差距，得到这个差距后，就可以使用DAX中的DATEADD来对去年同期数值进行时间段的偏移，此方案可以简称SQL方案。另一种方法是本文主要讲解的方法，完全使用DAX来处理此需求。

### 数据

本文使用微软提供的零售示例数据库[ContosoRetailDW](https://www.microsoft.com/en-hk/download/details.aspx?id=18279)，选区其中部分表建模如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091112053843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

前端报表布局如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911162614467.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

### 实践

##### 方案一

对于SQL方案，此处简单带过。首先构造一张表，用以展示不同年份和上一年相比的星期数偏移值：

>```SQL
SELECT A.[Datekey]
      ,DATEADD(MONTH,12,A.[Datekey]) AS [Date_NY]
      ,DATEPART(WEEKDAY,A.[Datekey]) [Week]
	  ,isnull(DATEPART(WEEKDAY,B.[Datekey]) - (CASE WHEN DATEPART(WEEKDAY,A.[Datekey]) > DATEPART(WEEKDAY,B.[Datekey])  
	  THEN (DATEPART(WEEKDAY,A.[Datekey])-7) 
	  ELSE DATEPART(WEEKDAY,A.[Datekey]) END),0) Week_Diff
      ,A.[CalendarYear]
  FROM [ContosoRetailDW].[dbo].[DimDate] A
  LEFT JOIN [ContosoRetailDW].[dbo].[DimDate] B
  ON DATEADD(MONTH,12,A.[Datekey]) = B.[Datekey]
  where DATEPART(MONTH,A.[Datekey]) = 1 and DATEPART(DAY,A.[Datekey]) = 1
>```

如下，Week_Diff即偏移值：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911151450682.png#pic_center)

将此字段使用RELATED函数新增至主表，然后在计算去年销售额时使用DATEADD应用日期偏移即可。

##### 方案二

在日期表新建两列，周数和星期数：

>```SQL
>WeekNum = WEEKNUM ( 'DimDate'[Datekey], 2 )
>
>----------------------------
>
>WeekDay = WEEKDAY ( 'DimDate'[Datekey], 2 )
>```

回到主表，新建列WeekCode用以为数据集做第几周星期数的唯一标识：

>```SQL
WeekCode = 
CONCATENATE ( RELATED ( DimDate[WeekNum] ), RELATED ( DimDate[WeekDay] ) )
>```

新建一列表示去年日期：

>```SQL
LY = DATEADD('DimDate'[Datekey],-12,MONTH)
>```

最后，新建计算列SalesAmount LYSW来表示去年同期最近星期数的销售额：

>```SQL
SalesAmount LYSW = 
CALCULATE (
    SUM ( FactSales[SalesAmount] ),
    FILTER (
        'FactSales',
        AND (
            AND (
                'FactSales'[StoreKey] = EARLIER ( FactSales[StoreKey] ),
                'FactSales'[WeekCode] = EARLIER ( FactSales[WeekCode] )
            ),
            AND('FactSales'[DateKey] < EARLIER ( 'FactSales'[DateKey] ) ,
                FactSales[Datekey]> EARLIER ( 'FactSales'[LY]) )
        )
    )
)
>```

该公式的意思即是寻找同店铺在上一年的区间内的相同周数且相同星期数的行集，并在此行集内求出销售额。将此字段放置到前端如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911171112577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

按上图选择的日期范围内（2019年全年），SalesAmount LYSW即表示在2008年1月3日（周四）至2009年1月1日（周四）内的销售额，利用Excel验证数据无误。

### 拓展

以上是我们利用DAX解决非常规时间智能问题的一个典型案例，实际业务也会遇到更复杂的场景，比如考虑到零售行业关店的场景，如何在同比计算中，排除本年和去年所有关店日期的销售额，假设本年第一周的周三因关店无数据，在与去年进行同比时，需要排除掉去年第一周的周三的数据，同样，如果去年的周三无数据，也需要排除今年第一周周三的数据，这样对比才更加科学合理，这些计算既可以在后端通过SQL处理，也可也利用DAX实现。遇到问题不要轻易问人，多思考，多想想不同的可能性，很多问题都能迎刃而解！