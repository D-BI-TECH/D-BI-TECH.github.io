---
layout: post
title:  PBI Report Builder 简介
date:   2019-05-15 06:03:50 +0000
image:  04.jpg
tags:   [Power BI,SSRS]
---

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515153758883.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

<big>**一、简述**

微软在上个月发布了Power BI Report Builder (*以下简称Report Builder*), 该应用主要用于构建分页报表，并将作为整个Power BI体系的重要组成部分, 如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515153831962.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

事实上，Report Builder的前身SSRS(*SQL Server Reporting Services*)已经发布很多年, 其与SSRS 2016的界面及功能几乎一样，反而Report Builder还有不少功能被阉割了，支持的数据源也更少了，这一方面是因为，Report Builder将作为一款专为Power BI Service 而优化的分页报表开发工具，而不是针对于发布在本地部署的服务器的SSRS, 因此部分功能需要做出调整，另一方面是因为有很多尚待开发的功能没有完成。尽管Report Builder作为一款"过时"的报表开发工具，其未来的创新还是可期的。

<big>**二、相比于Power BI Desktop的主要优势**

微软官方的说法，是PBI Report Builder更适合制作用于打印的正式报表, 但除此外，它还有许多其他优势。

**1.拥有更强大的表格数据交互能力**
在Report Builder中，你可以针对于每个单独的度量值、列标签以及行标签设置可以到达不同子报表的数据下钻，以及设置不同的URL和书签：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515153907892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

而在Power BI中，除了无法通过点击表格中的某个数据到达指定页面或书签，也无法做到在同一个表格控件中针对于不同的数据设置不同的子报表钻取（*尽管截至今年4月Power BI desktop已经可以做到跨报告级的数据钻取，但是这种钻取只能基于整个表格控件统一设置子报表，再通过钻取级筛选器筛选钻取后的对应结果，和Report Builder的钻取能力还有差距*）

**2.报表发布后其数据可以导出成多种格式**
其中包括：Word ,Excel, PPT, PDF, CSV, XML等，而Power BI报告发布后暂时只能导出CSV和PDF格式；值得一提，在Report Builder设置的URL和数据钻取即使导出到Excel其链接功能依然可用

**3.可以读取存储过程（*Stored Procedure*）**
Report Builder不仅可以使用SQL获取数据库数据，而且还可以调用存储过程，把获取的数据作为报表的数据集，Desktop目前无法做到。这就意味着用户可以在报表前端来改变数据查询的行为，比如用户在报表前端选择不同的月份，或者Group By到不同的维度，分页报表就会将这些选项作为参数，影响存储过程的执行，从而使数据库返回对应的查询结果，而无需加载整个数据集，因此理论上具有更好的性能. 此外可以利用存储过程实现数据的增量抽取(*Power BI 也可以，只不过需要购买License*)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515153924243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

**4.发布到Power BI Report Server后，可以使用邮件订阅的方式把报表分享给组织成员**
（*如果是使用Power BI Desktop做出的报表，在此处不具备邮件订阅功能*）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515153946716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)


<big>**三、简明教程**

**1.连接数据源**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019051515400143.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

**2.配置数据集**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515154013732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

**3.设计报表**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515154023248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

**4.运行报表(点击[home]--[Run])**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190515154032581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

**5.发布报表**
截至目前的2019年4月版本的Report Builder尚未提供发布到服务器的选项(*期待未来的更新*), 但用户可以在Power BI Service把Report Builder报表文件上传到Premium空间，也可以直接上传到Power BI Report Server
(*本文以介绍为主，使用教程不做详细介绍，Power BI Report Builder截至今年5月15日还没有官方介绍文档，详情可参考此SSRS Report Builder教程*）

<big>**四、其他**

就目前而言，相比于Power BI Desktop, Report Builder的劣势明显，M语言在此处毫无用武之地，内置公式库相比Excel还要少许多，数据处理很大程度上依赖于SQL技能，较弱的可视化能力以及更少的数据源支持, 至于DAX, 你也仅仅只能用它作为一种专用于查询SSAS表格模型数据库的查询语言。但不管怎么说，微软既然已经把Report Builder加入到了Power BI大家庭，就说明它还有发展前景和上升空间，以及它相比于Desktop的独特优势也满足了特定用户的需求。

(*注：本文同时发布于CSDN,知乎,现已迁移至本站*）