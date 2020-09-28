---
layout: post
title:  Power BI REST API有多强大？PBI开发者必读
date:   2020-09-28 01:03:50 +0000
image:  13.jpg
tags:   [Power BI,REST API]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*本文是D-BI之Power BI REST API系列第一篇，主要讲解Power BI REST API的概念，功能以及意义。后续第二篇和第三篇将讲解Power BI REST API的具体调用方法*

### 什么是Power BI REST API

在解释Power BI REST API（下文简称PBI API）之前，先理解何为REST API。API是应用程序与其他应用程序通信的一套规则，而REST（Representational State Transfer，中文：表现层状态转换），通俗而言即为开发人员在创建API过程中遵循的一组规则，或者说是一种风格，其优势是简洁以及较好的兼容性。**Power BI REST API既可以理解为专用于Power BI Service, 符合REST风格的Web API服务.** 使用该服务，即利用URL的方式向服务器发送一组请求以实现具体功能（比如获取Power BI工作区内所有数据集信息）。一般而言，请求由Endpoint（终结点），方法，请求头以及主体组成，其中请求的方法其一般包括GET,PUT,POST以及DELETE四种请求方法。![图片来自phpenthusiast.com](https://img-blog.csdnimg.cn/20200928165124502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)



### PBI API能做些什么

若不考虑您账户权限问题，则PBI API主要可以总结出如下几大用处。

1.获取您Power BI Service中所有工作区，数据集，数据流，报表以及仪表板等信息


2.向指定的Power BI数据集的指定数据表按行增加或删除数据，甚至可以创建一个新的数据集

![图片来自docs.microsoft.com](https://img-blog.csdnimg.cn/20200928164142373.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

3.修改您Power BI Service中任意工作区，数据集，数据流，报表以及仪表板等相关设置

4.在Power BI Service内执行一个或多个任务，如刷新数据集

5.为一个或多个Power BI报表和数据集创建嵌入令牌，以便报表可嵌入至其他应用程序内

![图片来自docs.microsoft.com](https://img-blog.csdnimg.cn/20200928164628176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

### PBI API在企业BI架构中的意义

在了解过PBI API的主要用处后，它对于企业BI架构的最重要意义是什么呢？是可以利用API让报表实现动态的数据写入？还是允许我们自定义PBI数据集的刷新时间？

事实上，这些还仅仅是表面上的意义，**Power BI作为一款微软生态下的商务智能解决方案，如果它可以与我们自己企业的现有的BI体系紧密结合，将Power BI服务纳入企业自有PaaS架构中，那么数据的抽取转换，刷新以及业务分析，知识提取等将形成一个更加稳固的链条，Power BI将不再成为企业数据中台中独立出去的一套体系**。利用PBI API，开发者可以将针对于Power BI Service的一系列操作集成到企业宏观体系的数据架构内。举一个冰山一角的例子，业务人员可以在企业自有的数据中台，向其有权限的PBI报表修改产品的折扣数据以进行临时性分析，修改可以立即启动数据刷新且完全不影响后台数据。

此外，Power BI Service中所有工作区，数据集，数据流，报表以及仪表板等信息的获取，甚至部署流程的获取，都意味着企业对PBI的管理能够更加高效灵活。当然，再好的技术，离不开企业管理层的管理模式和重视程度，也离不开各部门的配合程度和项目执行力度，作为BI管理者或工程师，我们能做的既是利用Power BI REST API让我们的数据流更高效，更准确，让报表尽可能发挥出数据的价值。
