---
layout: post
title:  PBI Report Builder 系列：报表发布
date:   2020-03-21 08:03:50 +0000
image:  03.jpg
tags:   [Power BI,分页报表,Report Server]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---


本文主要讲述Power BI Report Builder 报表的发布（本文默认您已拥有一份自己开发的分页报表或已阅读过本系列此前的章节）。

### 内容概览

本文主要讲解以下内容：

1. 云端部署：将分页报表发布到Power BI Service
2. 本地部署：将分页报表发布到Power BI Report Server
3. 混合部署：将本地部署的分页报表钉选到Power BI Service 仪表板

### 云端部署

如需将分页报表发布到Power BI Service,只需选择另存到Power BI Service即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200320230049603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

但可惜的是，截至目前（2020年3月），Power BI Service对分页报表的支持仅限于Premium用户，如果你的Power BI账户是免费版或Pro版，你将在发布时收到下图报错：
（已经有接近四千人投票，希望分页报表能够在Pro  Lincense账户上得到支持了，你可以[点此](https://ideas.powerbi.com/forums/265200-power-bi-ideas/suggestions/35959420-paginated-reports-please-make-it-available-in-pr)参与投票）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200320225903424.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

为了提供实操演示，我特地到Azure开启试用了Power BI Premium，如下图，我的Power BI工作区已经是Premium工作区了（你可以看到钻石标识），但依然在发布时遇到了错误，报错信息提示我们需要为该工作区开启分页报表工作负载：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020032101073325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

通过查阅官方文档，意识到了并不是所有Premium账户都可以将分页报表托管在Power BI Service（准确说是Azure Cloud）的残酷事实，只有Power BI Premium SKU A4及其以上才能在工作区开启分页报表工作负载，如下所示SKU A4，接近六千美元，不是每年，而是每月，对于大多数企业而言都是一笔不小的开销：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321012430815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

因此，我只好把原来的SKU A1（默认选项）修改为SKU A4的版本，现在终于可以将分页报表发布到Power BI Service了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321020836427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

对于使用本地数据库作为数据源的分页报表，记得提前配置好网关。在工作区打开分页报表，如下图所示，现在，我们已经成功实现在Power BI Service上使用分页报表了！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321021359584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 本地部署

将分页报表发布至Power BI 报表服务器，只需要在报表服务器Web门户直接上传报表文件即可：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321170856229.png)

你也可以直接在Report Builder中发布报表，但这需要你使用SSRS版的Report Builder, 而非Power BI Report
 Builder. 在SSRS版Report Builder，你可以使其连接到报表服务器，然后你就可以在Report Builder直接将报表另存到服务器中。你还可以选择"Publish Report Parts"发布报表中的单个图表，以供他人或后续重复利用。

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321171131469.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 混合部署

 混合部署模式也可以称为本地集成云部署，在该模式下，分页报表同样是部署在本地的报表服务器，但你可以将部分本地报表钉选到Power BI Service仪表板，定期刷新。不幸的是，这里存在很多限制，你只能将一些图表（比如柱状图）钉选到仪表板，但无法将分页报表真正具备优势的表格或矩阵钉选到仪表板，详情可以参考[此文档](https://docs.microsoft.com/en-us/sql/reporting-services/pin-reporting-services-items-to-power-bi-dashboards?view=sql-server-ver15#bkmk_supported_items)。
   
 实现方法是，打开报表服务器配置管理器，在“Power BI服务(云)选项卡中注册Power BI：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321174445702.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 回到Power BI Report Server门户网站，在”我的设置“处登录Power BI，这样你就可以在打开分页报表后看到钉选到Power BI的标志：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321174828317.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 然后你将可以设置最高每小时一次的更新频率：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200321174944917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 点击固定后，你就可以在Power BI Service上看到该分页报表的磁贴了，混合部署至此完成。

### 其他

 至此Power BI Report Builder的入门文章系列已画上句号，受限于本人时间精力，只讲了核心内容，对分页报表的功能细节未能详尽。巧合的是，近日微软官方刚好推出了[Power BI Report Builder 视频教程](https://docs.microsoft.com/zh-cn/power-bi/paginated-reports/paginated-reports-online-course)（在YouTube发布)，有条件的读者可以去下载观看，但我更推荐的是，你在实际的使用中，根据教程结合自行探索来完善你的分页报表知识体系。




#### 【附】系列目录
 - [x] [序篇]({{site.baseurl}}/pbi-reportbuilder-series/)

 - [x] [导入（配置）数据集]({{site.baseurl}}/pbi-reportbuilder-dataset/)

 - [x] [报表设计]({{site.baseurl}}/pbi-reportbuilder-design-tutorial/)

 - [x] 报表发布（本文）
