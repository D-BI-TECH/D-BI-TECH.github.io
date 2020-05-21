---
layout: post
title:  PBI Report Builder 系列：序篇
date:   2020-03-07 02:03:50 +0000
image:  14.jpg
tags:   [Power BI,分页报表]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

### 关于发表分页报表文章系列的说明

前些日子，我和文超老师有过简短讨论，我们都认可分页报表在财务以及大型数据集项目领域有着不可替代的优势。过去，人们喜欢拿Power BI和Tableau做对比，因为它们之间有明显的同类竞争关系，都属于交互性较强的敏捷BI开发工具, **但Report Builder（即[分页报表](https://docs.microsoft.com/zh-cn/power-bi/paginated-reports/paginated-reports-report-builder-power-bi)）却不同，它和Power BI报表实际上是互补的关系。** 分页报表做不到Power BI那样强大的交互展现能力，DAX引擎的高效率，以及利用PowerQuery对源数据的灵活处理；但另一方面，Power BI报表也做不到分页报表那样格式更加规范，打印更美观，拥有以更多文件格式，无任何行数限制以及所见即所得的像素级报表导出能力。

一年前，在Power BI Report Builder 发布不久，我曾发表了一篇[关于该工具的完整介绍](https://d-bi.gitee.io/introduction-pbi-reportBuilder/)，是当时国内外最早一批讲述PBI Report Builder的文章（当时微软还没有发布关于此的官方文档），对于该报表工具，业内普遍的理解是：一款可以发布到Power BI Services的SSRS Report Builder，确实如此，但在近一年的数次版本迭代后，该工具作为Power BI Services体系的一部分已渐趋成熟。过去，人们的关注点都集中在Power BI报表领域，鲜有人关注并了解分页报表，在新一轮的自助式BI浪潮前，传统BI显得有些落寞。但根据我最近的观察，国外已经有小部分用户开始主动关注分页报表了，重要的是，这部分人并非传统意义的IT开发者，而是新兴的Power BI用户，传统的BI外壳已经开始注入了新的活力。因此对于国内BI用户而言，无论你服务于小公司还是大企业，对分页报表仅仅局限于了解已经不够，你还要掌握如何去使用它。

![图片来自Microsoft Power BI Blog](https://img-blog.csdnimg.cn/20200307173259561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

<small>(微软官方博客提供的分页报表效果图，仅供参考）</small>

### 本系列文章目录

基于以上的原因，我决定推出分页报表文章系列，这个系列很短，而且完全是面向小白的入门级内容（见尾注）。该系列一共只有如下几篇：

**1. 序篇**

即本篇，系列说明与内容概览
    
**2. 导入（配置）数据集**

事实上在Report Builder不存在导入数据的概念，此部分实际上是使报表和数据源建立连结关系，定义数据集的查询语句，这里讲”导入数据集“是为了方便Power BI或PowerPivot用户的理解，因为在Power BI，通常都是使用Import模式，首先把数据导入到报表。而分页报表连接数据源类似于Power BI的DirectQuery，只有用户在使用报表的时候才会执行查询，因此同样也不存在数据刷新的概念，因为返回的结果就已经是最新的数据了。具体操作会在此文中进行说明。
    
**3. 报表设计**

这是本系列的**核心部分**，一步步带你完成你的第一个分页报表，包括各种格式调整以及可视化。

**4. 报表发布**

如果你已阅读前面的部分，那么最后一步就是发布报表，这里存在两种发布方案，本地发布以及云端发布，都将会在文章中进行演示。


### 后续内容

除以上篇章外，后续可能还会不定时发布关于分页报表的知识，比如报表下钻，子报表设定及特殊可视化的设定等进阶内容。但也有可能**取消**这部分内容，这一切取决于上述篇章的阅读量及关注度。

*（尾注：D-BI不会总是发布面向进阶读者的文章，但会尽量避免发布别人已经写过的内容，决不做重复性劳动。坚守原创性和精简性是本站的核心价值，相信你会在D-BI **"See Something Different，See Something Valuable"**）*


#### 【附】系列目录
 - [x] 序篇（本文）

 - [ ]  导入（配置）数据集 

 - [ ] 报表设计

 - [ ] 报表发布
