---
layout: post
title:  Power BI Report Server 连接共享数据集
date:   2020-12-13 00:03:50 +0000
image:  23.jpg
tags:   [PBIRS,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*本文讲述使用Power BI Desktop (rs) 连接RS共享数据集的方法*

### 前述

写几句题外话。细细数来，本文刚好是我在PBIRS领域发布的第十篇博客。还是那个原则，(至少在中文社区) 绝不发别人重复过的内容。因为我不知道这样做除了为自己吸引流量之外，对整个技术社区的发展有何意义。当然，转载文章或翻译国外文章还是有很大意义的，但如果不注明原文来源，盗为己用，那么引用前国脚范志毅的一句话来说就是： “脸都不要了”。

### 关于共享数据集

共享数据集，即托管在报表服务器上的SQL查询或存储过程，它可以被一个报表或多个报表共同利用，且可以被组织内其他的报表开发者使用。由于Power BI Report Server是SSRS 2017的超集，因此它也能够支持共享数据集。如下，你可以在PBIRS Web Portal上使用Report Buider来创建数据集：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213003626286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

通常而言，共享数据集基本用于分页报表（Paginated Report）,因为该功能原本就是为了分页报表设计的，这一点很多人都了解。但Power BI报表可以连接共享数据集吗？这一点知道的人并不多，对此官方文档提供的资料也十分简略，事实上，利用REST API，我们可以使用开放数据协议（Odata）使Power BI报表连接到共享数据集。

### 使用Odata连接共享数据集

过程十分简单，仅需两步。

第一步，找到我们需要连接的数据集的ID。我们可以向PBIRS发送POST请求获取。此处，直接使用浏览器访问以下网址，所有完整的数据集信息将以JSON形式返回：

```SQL
http://<YOUR PBIRS URL>/Reports/api/v2.0/datasets
```

第二步，找到我们需要连接的数据集ID后，打开Power BI Desktop (RS), 选择Odata作为数据源：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020121301011774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

输入以下链接并勾选“Include open type columns”：

```SQL
http://<YOUR PBIRS URL>/Reports/api/v2.0/datasets(<YOUR SHARED DATASET ID>)/data
```

或直接在高级编辑器输入M语句：

```SQL
= OData.Feed("http://<YOUR PBIRS URL>/Reports/api/v2.0/datasets(<YOUR SHARED DATASET ID>)/data", null, [Implementation="2.0", MoreColumns=true])
```

运行查询后，只需展开列即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213012230669.png)

### 问答

##### 1.使用共享数据集的Power BI报表可以正常数据刷新吗？是否支持DirectQuery？

可以设置数据刷新计划，但只能是导入模式，不能直连。

##### 2.Power BI可否读取使用存储过程建立的共享数据集？

可以。但需要为存储过程指定好默认参数，因为Power BI不能像分页报表一样可以允许用户在前端实现向后端的动态传参。

*标准版（云端）Power BI 已在今年10月底推出了[动态M语句](https://docs.microsoft.com/en-us/power-bi/connect-data/desktop-dynamic-m-query-parameters)预览功能，但局限性较大，关于此，后续会考虑写博客介绍*

##### 3.对于Power BI报表，共享数据集有什么意义，或者说有哪些适用场景？

共享数据集的一个重要作用，即是把后端（建立数据集）的任务从整个报表开发流程中分离出来，这对于完整的BI团队的分工协作有重要意义，后端不需要花时间在报表可视化的设计上，前端也不需要构建复杂的SQL查询或存储过程，而是把精力集中在模型关联，建立DAX表达式与可视化设计。

其次，同一个共享数据集可以用在多个报表上，包括Power BI报表与分页报表，因此可以大幅节省后端数据集维护的时间成本。




