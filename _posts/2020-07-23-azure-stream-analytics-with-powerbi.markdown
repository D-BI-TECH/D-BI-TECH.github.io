---
layout: post
title:  利用Azure Stream Analytics构建实时BI报表
date:   2020-07-23 01:03:50 +0000
image:  03.jpg
tags:   [Azure,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

*本文主要讲解利用Microsoft Azure中的Stream Analytics 作业服务（以下简称ASA），实现数据从多个数据输入源（比如终端设备，应用程序，传感器等等）到数据库或数据集的实时传输。本文将以虚拟的树莓派设备为例，实现数据向Power BI数据集的实时传输，最终利用Power BI实现数据可视化。*

### ASA 介绍

ASA，全称Azure Stream Analytics（Azure 流式分析），是Microsoft Azure提供的一项完全托管服务，引用官方文档的介绍：“Azure 流分析是一个实时分析和复杂事件处理引擎，旨在同时分析和处理来自多个源的大量快速流式处理数据。 可以在从许多输入源（包括设备、传感器、点击流、社交媒体源和应用程序）提取的信息中识别模式和关系。 ”，总的来说，即是一项能够帮助企业实现数据从终端设备或程序向数据库或数据集进行实时传输的一项数据流服务，其核心是实时传输，其理念是平台即服务。

*注：使用ASA需要拥有Azure订阅，作为学习和测试目的，可以创建并使用[Azure免费账户](https://azure.microsoft.com/en-us/free/)。国内用户也可以在[Azure中国区--世纪互联](https://azure.microsoft.com/zh-cn/free/)申请试用，但本文将以国际版为例进行讲解*

### Azure 实时流的实现过程

ASA作业主要分为三个部分，输入，查询以及输出。ASA可以从Azure Event Hub，Azure IoT Hub以及[Azure 存储账户](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview)实例中引入设备或应用程序发送的数据。在查询阶段，可以利用SQL对数据进行简单处理，并从输入端将数据INSERT至输出端。在输出部分，你可以配置数据传输的目标，它提供了如下输出端，这些都是基于云端的服务：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072216445986.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

结合本文主题，ASA作业的整体实现流程如下，其中实线的部分将是本文提供的方案：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722164912797.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)



### 创建 Azure IoT Hub 实例

创建这个实例的目的是用于接收设备或应用程序传递的数据。登录[Microsoft Azure](https://portal.azure.com/), 找到Azure IoT Hub后点击创建：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722170039906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

在这里你可以配置你的IoT中心的访问终结点权限（公开或者私密），甚至可以配置IP网段：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722170145895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

在此处选择如下配置即可。作为学习或测试用途的，后续可以随时删除实例以避免继续产生费用。创建完成后将会执行部署，通常一分钟左右即可完成。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722170316965.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

部署完成后，在IoT Hub新建一个设备，这个设备就是用于我们后续实现实时数据流的测试设备：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722171341275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

新建设备完成后，需要在此处记下主要密钥和主要连接字符串以备后用，如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722170730667.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 创建并配置ASA

返回Azure主页，找到Stream Analytics jobs后点击创建:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722171634409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

创建成功后即可在ASA中分别配置输入和输出。在输入部分，选择我们刚刚创建的IoT Hub：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072217182822.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

输出部分选择Power BI,你也可以根据你的需要选择其他目标。此处需要你提供Power BI的账户验证，通过后右上角会提示连接测试成功。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722172157288.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

*注：若后期该Power BI账户的密码被修改，你需要重新回到此处更新验证（如下图），否则流分析作业可能会提示错误*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722175147186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

最后，配置查询。按下图在SQL中指定好输入和输出端保存即可。后续你可以根据自己的需求更改SQL查询。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722173206818.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 启动ASA

在启动ASA之前，需要先启动设备，因为如果没有启动设备，系统会提示输入端没有任何数据，将无法启动ASA。本文以树莓派模拟器为例。[点此](https://azure-samples.github.io/raspberry-pi-web-simulator/)进入树莓派模拟器页面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072217400681.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

按上图提示操作后点击下方的运行键，将会有源源不断的数据不断输出，左侧红灯将会闪烁，至此设备启动成功。

现在回到ASA主页，点击启动：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722174558623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

启动成功后，数据流将会实时运行查询，不断将新的数据从输入端（IoT Hub) 传输至输出端（Power BI数据集）。至此我们已经完成了80%的任务，最后一步是登录Power BI创建实时报表。


### 利用 Power BI 生成报表

如果前面的一切步骤都顺利，现在回到Power BI, 你应该可以看到ASA为我们Power BI的指定工作区内创建了Power BI数据集（如图：Davis-Stream）, 用于存储数据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072217572793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

这样我们就可以直接利用数据集创建基于实时流数据的报表了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200722180024410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 总结

Azure Stream Analytics是真正意义上的，能为企业提供稳定且高效的实时流分析服务，相比Power BI Service提供的流数据集（即[PubNub方案](https://docs.microsoft.com/en-us/power-bi/connect-data/service-real-time-streaming)）而言要强大得多，它可以对接我们的数据仓库供后续使用，也可以像本文展示的，直接对接前端的BI报表服务，利用Power BI可视化数据，提供灵活的即席分析。更强大的，还可以利用Azure管道通过CI/CD 的方式部署ASA，并与Azure机器学习深度集成。
