---
layout: post
title:  关于 Azure Data Factory 及其优势
date:   2020-08-02 01:03:50 +0000
image:  05.jpg
tags:   [Azure,ETL]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

### 什么是 Azure Data Factory

Azure Data Factory （以下简称ADF），即Azure数据工厂，是一个部署于云端的数据集成系统，它允许在本地和云端之间转移数据，创建和编排复杂的数据流，并制定全面托管的作业，以实现自动的，按期执行的，无需人为干预的数据流。 按照官方的说法，ADF是一个“无代码ETL即服务”，可实际上，它更像是一个ELT平台 (提取-加载-转换）。但如果你熟悉SQL Server的生态，也可以简单地把ADF看成是云端版本的SSIS。


### 基本概念

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729233509210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)


1.**Linked Services （连接服务）**

包含数据源的配置信息及凭据。

2.**Activities （活动）**

活动代表动作，这些动作可以是数据移动，转换或控制流动作。 活动配置包含数据库查询，存储过程名称，参数，脚本位置等设置，活动可以获取零个或多个输入数据集并产生一个或多个输出数据集。 尽管可以将ADF活动与SSIS数据流任务组件（例如聚合，脚本组件等）进行比较，但是SSIS具有许多ADF尚不匹配的组件。

3.**Pipelines（管道）**

管道是活动的逻辑分组。 数据工厂可以有一个或多个管道，每个管道将包含一个或多个活动。 使用管道使计划和监视多个逻辑相关的活动变得更加容易。

4.**Dataset（数据集）**

包含数据源配置设置，但级别更细。 数据集可以包含表名或文件名，结构等。每个数据集都引用某个链接服务，而该链接服务确定可能的数据集属性列表。 链接服务和数据集类似于SSIS的数据源/目标组件，例如OLE DB源，OLE DB目标，除了SSIS源/目标组件在单个实体中包含所有特定于连接的信息。

5.**Integration runtime**

Integration runtime（IR）是ADF使用的计算基础结构，用于跨不同网络环境提供数据移动和计算功能。它包括：
   - Azure IR。 Azure集成运行时在Azure中提供了完全托管的无服务器计算，这是云中数据移动活动背后的服务。
   - Self-hosted IR。 该服务管理私有网络中云数据存储和数据存储之间的复制活动，以及HDInsght Pig，Hive和Spark等转换活动。
   - Azure-SSIS IR。 SSIS IR是本地执行SSIS包所必需的。


### 关于参数

在ADF中，可以像SSIS一样在管道中设置带有变量的数据流。它的作用可以用一个简单的例子说明。假设我们有一个Excel文件存储在本地计算机，或者如图示存在Azure 存储容器中，你需要把该文件的数据同步到Azure SQL 数据库，只需要在输出端和输入端分别设置一个数据集即可。但如果需要传输的文件有很多，那么使用变量可以允许你依然只需设置一次数据集。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729233632645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 相比于SSIS的主要优势

 1. SSIS通过SSMS管理，而ADF通过Azure门户管理，与包括AAD在内的其他Azure服务深度集成，拥有更高效的管理界面。
 2. SSIS是一个桌面工具（通过SSDT或Visual Studio），需要一台专用服务器，且需要日常维护。ADF是一项基于云的服务，属于平台即服务（PaaS）工具，因此不需要硬件或进行任何安装。 
 3. SSIS则不具备“数据沿袭”，而ADF则具备此特性，它可以标记和跟踪来自不同来源的数据。 
 4. SSIS是典型的ETL工具，而ADF在功能上既可以作为ETL工具，也可以作为ELT工具，因为ADF同样可以使用函数，变量，参数，以及SSIS上的一些关键组件（比如Foreach），并且可以执行SSIS包，此外，在Azure平台可以利用其他的服务来完成数据的转换任务（比如AAS等），ADF只需要完成抽取和加载，这种ELT模式也已经逐渐取代ETL成为了企业BI的发展趋势（这是另一个话题）。
 5. ADF支持更多的开发方式进行创建任务及部署，包括C#，Python，REST API等等。
 6. ADF可以与GitHub或Azure Devops集成，这允许您在开发，构建任务时自动部署到Azure。
 
 *注：相比于ADF，SSIS主要在数据转换方面的功能更加强大，有更加齐全的功能和组件。然而，由于你依然可以在ADF上执行SSIS包，因此对于ADF而言同样是优势，你还可以利用[Azure 数据库迁移服务](https://docs.microsoft.com/zh-cn/azure/dms/how-to-migrate-ssis-packages-managed-instance)将 SSIS 包迁移到 Azure SQL 托管实例*
 
### 后续内容

本文是对Azure ETL服务ADF技术的开题篇，你可以阅读[ADF官方文档](https://docs.microsoft.com/zh-cn/azure/data-factory/)以系统性学习，此外本站未来还将推出1~2篇关于Azure数据工厂的详细使用和部署教程，将会包括基础的数据同步和数据流的建立，以及部分官方文档未覆盖到的操作问题，敬请期待。



