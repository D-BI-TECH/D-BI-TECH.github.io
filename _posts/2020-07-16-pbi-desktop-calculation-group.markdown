---
layout: post
title:  Power BI Desktop 关于使用计算组
date:   2020-07-16 06:03:50 +0000
image:  02.jpg
tags:   [Power BI,Tabular Editor]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---


*本文是对PBI Desktop新功能计算组的一点补充说明。具体理论和应用，请参考文末所列的文章及资料，我不会做任何关于此的重复劳动，这些资料已经非常完美。*

### 计算组在Power BI Desktop已获支持

昨天，微软Power BI和往常一样，发布了Desktop的[月度更新](https://powerbi.microsoft.com/en-us/blog/power-bi-desktop-july-2020-feature-summary/)，不同以往的是，这次更新不仅仅新增了一部分来自Excel的金融类DAX函数，最令人惊喜的是，这个版本的Power BI表格模型已经潜在地支持了计算组。

*注：在旧版本Power BI Desktop，比如最近的2020年6月版本，尽管它的模型兼容级别一样是1520，但你无法强行为数据模型添加计算组，即使按要求将下图的参数值设置为true. 如图：*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716180037566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

事实上，该功能在半年前就已经发布，并且新增了计算组的所有配套DAX函数，然而，和微软Power BI在最近一年发布的其他黑科技一样，这仅仅面向Premium 用户，因为Premium 用户可以利用XMLA终结点直接对表格模型进行写入（这使得Power BI Premium在一定程度上取代了Azure AS的地位，当然这是另一个话题了），利用这种方式来创建新的计算组。**但现在，它已不再是Premium 用户的专利了**，Pro 用户一样可以在Power BI Desktop中，利用外部工具Tabular Editor创建计算组（或者也可以直接在DataModelSchema文件中手动添加），并且在发布到Power BI Service后同样受到支持。


### 关于计算组的使用

半年前，在我发布的[《DAX：Calculation Group 概念与应用》](https://d-bi.gitee.io/dax-calculation-group-application/)中，讲解了在SSAS2019 + Power BI的方案下，计算组的具体概念和应用方法，当时期待在未来版本Desktop中能够提供一个创建计算组的操作界面，遗憾的是，我们可能不会在Desktop中看到这样的界面了，因为该操作已经以外部工具Tabular Editor的形式得到实现:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716164419548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

打开Tabular Editor后，该工具会读取PBIX或PBIT的模型文件DataModelSchema，这样你就可以直接以界面窗口的方式新增计算组以及计算项，首先你需要注意两点：

1. 必须事前安装[Tabular Editor最新版本](https://github.com/otykier/TabularEditor/releases/tag/2.11.6)
2. 在PBI Desktop预览功能处启用[enhanced dataset metadata](https://docs.microsoft.com/zh-cn/power-bi/connect-data/desktop-enhanced-dataset-metadata)

确保上述两点后，你就可以正常地在Tabular Editor创建计算组了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716165515118.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

关于具体的创建操作，使用教程，SQLBI的Macro Russo已于昨天新版本发布当日发布了教程视频，我极力推荐你去观看该教程，对此Macro已经讲的十分全面了，但如果你不想花太多时间，下图将会是一个很好的参考。并且，我不得不说Tabular Editor对于新功能计算组的支持已经非常的完美了，我们已经没有必要修改模型文件DataModelSchema，因为他包含了所有的可用设置，包括设置不同计算组的优先级。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200716171959510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 更多参考

【2019-6-18】【英】[《计算组介绍》](https://www.sqlbi.com/articles/introducing-calculation-groups/)

【2019-6-17】【英】[《理解计算组》](https://www.sqlbi.com/articles/understanding-calculation-groups/)

【2019-7-11】【英】[《理解计算项的应用》](https://www.sqlbi.com/articles/understanding-the-application-of-calculation-items/)

【2019-7-18】【英】[《理解计算组优先级》](https://www.sqlbi.com/articles/understanding-calculation-group-precedence/)

【2020-2-9】【中】 [《DAX：计算组概念与应用》](https://d-bi.gitee.io/dax-calculation-group-application/)

【2020-3-26】【英】[《在计算组中控制格式字符串》](https://www.sqlbi.com/articles/controlling-format-strings-in-calculation-groups/)

【2020-4-17】【英】[《在PBI Premium及AAS设置计算组》](https://www.kasperonbi.com/adding-calculation-groups-to-aas-or-pbi-premium/)

【2020-4-21】【英】[《避免计算组优先级的陷阱》](https://www.sqlbi.com/articles/avoiding-pitfalls-in-calculation-groups-precedence/)

【2020-7-15】【英】[《使用Tabular Editor在Power BI Desktop创建计算组》](https://www.sqlbi.com/tv/creating-calculation-groups-in-power-bi-desktop-using-tabular-editor/)
