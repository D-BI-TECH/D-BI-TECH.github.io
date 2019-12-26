---
layout: post
title:  关于Power BI Dataflow
date:   2019-01-22 07:03:50 +0000
image:  01.jpg
tags:   [Power BI,Power Query]
author-name: Daniil Maslyuk
author-image: Daniil.jpg
---

<small>[原文](https://xxlbi.com/blog/power-bi-dataflows-considerations/)译注：Davis ZHANG  </small>

----------------------

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209004211453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

最近我把很多M查询从.pbix文件迁移到了Dataflows，本博文中，我将分享一些在此过程中的心得。

数据流是在线设计的Power Query，其输出作为CSV文件存储在Azure Data Lake Storage Gen2中。因此，在处理数据流时需要注意一些事情。我在处理数据流时遇到的一些问题，比如糟糕的UI，这些可能只是暂时的，我不会一一列出，因为它们可能很快就会消失。

在CSV文件中存储数据可能还需要考虑其他一些因素，在设计数据流时需要记住这一点。以下是我在这篇博文中谈到的问题：

1. 空字符串 vs NULL值
2. 压缩
3. 现有表关系
4. 数据类型

#### 空字符串 vs NULL值

Power Query确实区分了文本列中的空值和空字符串。这是它们在dataflows PowerQuery编辑器中的显示方式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209004254774.png)

问题是，当您使用数据流时，两者的存储方式是相同的：当您处理文本时，总是得到空字符串。这是从数据流获取Power BI Desktop中的数据时看到的情况：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209004306593.png)

**启示**

加载时，M中的空值变为DAX中的空值。获取空字符串而不是空值可能会导致意外结果，因为DAX中的不同函数对空字符串和空值的处理方式不同。

**Tips**

我个人认为空值比空字符串更可取。如果您和我一样，则在从数据流加载数据时，需要将空字符串替换为空值：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191209004317658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 压缩

如果您是Power BI的长期用户，您将知道它非常擅长压缩数据，因为它的引擎在幕后使用了一个名为VertiPaq的列式数据库。

如果有一个表只有一列，并且相同的值重复了一百万次，那么.pbix文件的大小将很小，更精确地说，大约是25KB。实际存储的值无关紧要-大小将是相同的，无论是“1”重复一百万次，还是“这是一句为了更你惊喜的超级重复的长句”重复一百万次。这是因为VertiPaq在压缩数据时使用了智能算法——此主题不在本文的讨论范围内。

与之相比，包含重复一百万次的相同值的CSV文件的大小与重复值的长度成正比，因为它根本没有被压缩。

**启示**

当您使用数据流时，您需要注意csv的压缩方式是完全不同的。我不是这方面的专家-我只知道即使数据被压缩，压缩也远不及Power BI。  
具体来说，使用长文本键（如GUIDs）的数据流比使用整数键的数据流占用的空间要大得多。即使您没有为数据流使用自己的存储也仍然会影响到您--刷新数据集需要更长的时间。因为要读取的文件大小（CSV）会更大。数据流本身也可能需要更长的时间来刷新（或写入）。

**Tips**

尽可能地缩短正在使用的值。例如，如果逻辑类型列的值为true和false，请将它们保存为整数--1表示true，0表示false。这将减少每个值3或4个字节，因为true值和false值的存储方式完全相同（true值和false值，占用4或5个字节）。在Power BI Desktop中读取数据流时，将数据类型设置为Logical，Power BI将正确地动态地将值从1和0分别转换为true和false。

另外，如果切换到数据流后.pbix文件的大小变大，也不要感到惊讶。在使用数据流之前，我的一个.pbix文件是150MB；之后，使用相同的数据流变成200MB。数据是一样的，我仍然不知道为什么会有这么大的差异。虽然我可以尝试对数据流进行排序以改进压缩，但当数据流CSV文件的大小为几GB时，这并不是一个理想的选项。

根据我的观察，Power BI Desktop首先读取数据流的一部分，这样它就可以使用它的启发式方法来确定压缩的最佳排序顺序。我最初的查询来自于一个数据库，所以也许Power BI曾经获得了一个更高质量的样本，并且在排序数据时做出了更好的选择 —— 希望将来我们能够更改默认设置并让Power BI花更多的时间找出最佳排序顺序，就像我们今天在Analysis Services中所做的那样。

#### 现有表关系

如果用dataflow替换现有的Power Query查询，请注意这样做可能会破坏现有表的关系。

**启示**

您不应该认为简单地用数据流替换查询就可以了。即使您维护的列名和类型相同，这些关系可能仍然会中断，您可能仍然需要重新创建它们。我不知道为什么有些关系能存活下来，有些关系却不能 —— 如果这个过程是可以预见的，那就太好了。

**Tips**

写下你的表关系列表，并在更新你的查询后检查它是否完好无损。为了简化任务，您可以使用VertiPaq Analyzer，它有一个列出所有关系的专用工作表。

#### 数据类型

我希望这个问题能解决。目前，Power BI desktop中的Power Query和Dataflow中的Power Query之间没有数据类型的对等性。具体为，

- Fixed decimal number 转换为 Double
- Date 转换为 DateTime
- Time 同样转换为 DateTime

**启示**

固定小数位数（Fixed decimal number）的问题仅仅是一个不便，因为这些数字无论如何都是用四个小数位存储的，所以您只需要在从数据流加载数据时设置数据类型。

有趣的是，如果下载dataflow的JSON文件，固定小数位数将是“double”数据类型，而常规的小数值将是“decimal”。Double是Power Query中与Decimal类似的合法数据类型，因此Power BI Desktop不将其读取为固定小数位数也就不足为奇了。

另一个有趣的逸事是，固定小数位数的数据类型在数据流中称为Currency — 一段时间以前Power Query中使用的数据类型的旧名称。

没有时间的日期也存储为日期，这是有效的。同样，您只需要在Power BI Desktop中加载数据时设置正确的数据类型。

时间被有效地存储而没有日期。加载数据时需要将数据类型设置为Time，否则将获得当前日期的时间，如22/01/2019 12:30 AM。

如果您觉得缺少对等校验很烦人，请在Power BI Ideas网站上投票赞成这个想法：Dataflow与Power Query数据类型对等校验。

**Tips**

连接到数据流时，请确保所有数据类型的设置都与预期的完全一致。不要仅仅因为在Power BI Service中正确设置了数据类型，就认为它们将被正确加载。

#### 总结

我真的很喜欢数据流。我相信微软很快就会缩小Power BI Desktop中的Power Query编辑器和Power BI service之间的差距，如果你记住Dataflow数据是以csv形式存储的，并且有其细微差别，你可以避免很多问题。

