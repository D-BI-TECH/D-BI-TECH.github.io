---
layout: post
title:  DAX函数：CONVERT和REMOVEFILTERS
date:   2019-08-23 06:03:50 +0000
image:  08.jpg
tags:   [Power BI,DAX]
author-name: Daniil Maslyuk
author-image: Daniil.jpg
---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191208170445417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

微软2019年8月发布了两个新的DAX函数：CONVERT和REMOVEFILTERS。它们非常新，在撰写本文时（2019年8月23日），它们仅在Azure Analysis Services和Power BI Service中可用，甚至[DAX Guide](https://dax.guide/)也没有列出它们。在这篇博文中，我展示了它们的用法。

#### CONVERT

![DAX CONVERT function](https://img-blog.csdnimg.cn/20191208173720669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

函数允许您显式地将表达式的数据类型转换为指定的数据类型。它接受两个参数:

1. 标量表达式
2. 数据类型

其中第二个参数--数据类型，可以是以下其中一种：

- BOOLEAN
- CURRENCY
- DATETIME
- DOUBLE
- INTEGER
- STRING

虽然我们以前可以用[CURRENCY](https://dax.guide/currency/)和[INT](https://dax.guide/int/)这样的函数进行数据类型转换，但这个函数是通用的，而且更容易记住。如果要将值转换为以某种格式显示的文本，则仍必须使用[FORMAT](https://dax.guide/format/)函数。

#### REMOVEFILTERS

![DAX REMOVEFILTERS function](https://img-blog.csdnimg.cn/20191208173753720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这几乎可以看作我们的老朋友--[ALL](https://dax.guide/all/)函数的新名字了。该函数只能在[CALCULATE](https://dax.guide/calculate/)中作为过滤器使用，它不能被用作表级别的表达式。

作为CALCULATE函数的过滤器，该函数的工作方式与ALL完全相同，并接受以下参数：

- 单独的表名
- 单独的列名
- 同表中的多个列
- 不含任何参数，留空

当你使用REMOVEFILTERS且不含任何参数时，它将忽略整个数据模型的筛选上下文。

从某种意义上说，REMOVEFILTERS与ALL的关系和RELATEDTABLE与CALCULATETABLE的关系完全相同：一个减少了功能的别名，它可以使代码更具可读性，这总是一件好事。

来自[《 Definitive Guide to DAX (2019) 》](https://www.sqlbi.com/books/the-definitive-guide-to-dax-2nd-edition/):

>当用作表函数时，ALL是一个简单的函数。它返回一列或多列的所有唯一值，或表的所有值。当用作CALCULATE修饰符时，它充当假设的REMOVEFILTER函数。

如今，该函数已不再是假设的了 🙂

你已经可以通过连接到发布到Power BI service的数据集来尝试此新函数了！

----------------------
由Davis ZHANG代译，[点此](https://xxlbi.com/blog/dax-functions-convert-removefilters/)查看英文原文

----------------------