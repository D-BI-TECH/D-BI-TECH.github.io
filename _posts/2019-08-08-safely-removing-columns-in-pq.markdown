---
layout: post
title:  在PowerQuery安全地移除字段
date:   2019-08-08 06:03:50 +0000
image:  11.jpg
tags:   [Power BI,Power Query]
author-name: Daniil Maslyuk
author-image: Daniil.jpg
level: 入门
---

<small>[原文](https://xxlbi.com/blog/safely-removing-columns-power-query/)译注：Davis ZHANG  </small>

----------------------

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191207164907155.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

在Power Query, 如果你要移除一个不存在的字段，你将遇到错误。尽管您可能不会故意删除不存在的列，但是当您从数据源中删除了该列时，就可能会发生此情况。

防止这种错误发生的主流方法是使用[Table.SelectColumns](https://docs.microsoft.com/en-us/powerquery-m/table-selectcolumns)函数，而非[Table.RemoveColumns](https://docs.microsoft.com/en-us/powerquery-m/table-removecolumns)，但当你要选择的列过多时，就可能使你的M查询代码过于冗长。在本文，我将展示另一种解决方案。

#### 案例数据

为了更好地说明该问题，我将使用仅有两个查询结果的简单数据集：

1. 源表
2. 转换表

#### 源表

这是包含原数据的最初查询结果，你可以把它想象成SQL Server中的视图或Excel中的一张表。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191207164946232.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

>```Python
let
    Source = #table(
        type table [A=number, B=number],
        { {1, 2}, {3, 4} }
    )
in
    Source
>```

#### 转换表

该查询在源表的基础上移除了B列。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191207164951837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)


>```Python
let
    Source = #"Source Table",
    #"Removed Columns" = Table.RemoveColumns(Source,{"B"})
in
    #"Removed Columns"
>```


#### 问题

如果我们使用上述查询，是没问题的，但当我们从源表移除B列后，将会引发错误：

>```Python
let
    Source = #table(
        type table [A=number, B=number],
        { {1, 2}, {3, 4} }
    ),
    #"Removed Columns" = Table.RemoveColumns(Source,{"B"})
in
    #"Removed Columns"
>```

现在我们看转换表的查询，我们将得到如下错误：

>Expression.Error: The column 'B' of the table wasn't found. Details: B

这个错误发生的原因是，我们指示Power Query去移除一个在源表已经不存在的列 -- B列。

#### 解决方案

为避免此错误发生，你只需使用[Table.RemoveColumns](https://docs.microsoft.com/en-us/powerquery-m/table-removecolumns)的第三个可选参数，用于处理丢失的字段。  
你可以使用MissingField.Ignore或MissingField.UseNull作为参数值（两种参数在此都实现了同样的效果）：

>```Python
let
    Source = #"Source Table",
    #"Removed Columns" = Table.RemoveColumns(
        Source,
        {"B"},
        MissingField.Ignore
    )
in
    #"Removed Columns"
>```

现在错误被排除了，且查询仅返回了A列 🙂

#### 一个想法

不幸的是，[Table.TransformColumnTypes](https://docs.microsoft.com/en-us/powerquery-m/table-transformcolumntypes)函数没有对应的参数去处理丢失字段，尽管在某些场景下这将十分有用。如果你支持该想法，请[点此](https://ideas.powerbi.com/forums/265200-power-bi-ideas/suggestions/31546837-include-missingfield-argument-with-table-transform)投票。

