---
layout: post
title:  Power BI Premium Per User (PPU) 介绍
date:   2020-12-22 01:03:50 +0000
image:  24.jpg
tags:   [Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*微软在上个月公开发布了Power BI Premium Per User （即高级个人版），这在技术圈内早已不是新闻，但今天还是想写篇短博做些介绍，一方面是多数国内用户对此还不了解，另一方面，它确实**很香**，值得一试。*

### 什么是Power BI PPU

在了解Power BI Premium Per User（以下简称Power BI PPU）之前，先过一下我们所熟悉的License。

- Power BI Free.
- Power BI Pro. 
- Power BI Premium.

我们知道，Power BI Free面向个人，它是免费的，你可以发布报表，但不能创建新的空间以及与同事一同协作，Power BI Pro则允许用户互相分享报表，创建App，并支持增量刷新，以及R可视化等新特性，它面向组织或企业，但按个人收费，价格便宜（每用户仅约10美元一月），而Power BI Premium则与前两者不同，它面向整个组织，在包含Power BI Pro全部特性的前提下，又多出了几项高级特性，比如部署管道，XMLA终结点等等，但它根据容量定价（其实就是配置），并按月收费，价格较为昂贵。

于是，我们就能发现，对于相当一部分群体，会面临尴尬的选择。他们希望拥有Power BI Premium的多数功能，但受限于组织规模以及预算，难以承受license的高昂价格，在这种情况下，Power BI PPU诞生了。

Power BI PPU是Power BI Premium的个人版，你也可以认为它集成了Pro的低成本以及Premium高特性的二者之优，结合[相关文档](https://docs.microsoft.com/zh-cn/power-bi/admin/service-premium-per-user-faq#general-questions)提供的信息，我整理了下表以展示他们之间的具体区别：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222225543656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

从上表来看，尽管Premium个人版并不完全拥有Premium的全部功能，但相比Pro而言，其所支持的几乎都是最关键的功能，比如部署管道，分页报表，甚至支持更高级的AI分析以及对于开发者至关重要的XMLA终结点，这已经包含除计算能力之外Premium的全部核心价值了。

##### 现在，几乎任何用户都可以免费试用Power BI PPU 60天!

对于已经订阅Power BI Pro的用户，可以在Power BI Service右上角启用Premium个人版试用：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222231634356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

对于免费用户（Power BI Free），也可以通过先试用Power BI Pro, 然后选择一个工作区来启用Premium 个人版：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222231910277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

这样你就能享受到PPU的全部功能了！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222232009106.png)

*注：如果已订阅Microsoft 365，管理员也可启用PPU试用；对于试用Power BI Pro过期的免费用户，则需要先升级到Pro。*

### 其他

目前Power BI Premium 个人版还处于公开预览阶段，且尚未公布定价，根据微软的安排，也许在明年的第一季度正式发布，希望能有一个合理的价格。更多的问题可以参考[此文档的问答部分](https://docs.microsoft.com/zh-cn/power-bi/admin/service-premium-per-user-faq#end-user-experience-questions)。

