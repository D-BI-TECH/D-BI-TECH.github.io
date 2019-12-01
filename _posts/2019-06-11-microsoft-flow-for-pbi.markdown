---
layout: post
title:  利用Microsoft Flow增强PBI使用效率
date:   2019-06-11 06:03:50 +0000
image:  04.jpg
tags:   [Power BI,Microsoft Flow,Office 365]
---

<small>*前述：Microsoft Flow和Power BI都属于微软Power平台的一部分。在微软Power平台的[官网](https://powerplatform.microsoft.com/en-us/)，你可以看到关于该平台的以下介绍：*<small>

>“Power BI、PowerApps和Microsoft Flow是为协同工作而设计的，因此您的企业中，无论其技术专长如何，每个人都可以快速轻松地构建自定义应用程序，自动化工作流以提高业务效率，并分析数据以获取洞察力。”

<small>*即是说，PowerApps和Microsoft Flow可以和Power BI互动，能够大大增强Power BI的拓展性。这其中，PowerApps可以允许用户无需任何代码即可构建符合其业务需求的自定义应用程序，你可以把Power BI DASHBOARDS上的数据磁贴嵌入到PowerApps中，同时，你也可以在Power BI利用PowerApps可视化控件，使用来自你在PowerApps构建的应用程序，可见PowerApps能够为Power BI带来更加强大的能力。但本文的重点在Microsoft Flow上面，它能够使你的业务流程自动化以大大提高你的工作效率，并且，这不是一个为Microsoft Flow做全面性介绍的博文，而是主要讲解如何利用Microsoft Flow更好地服务我们的Power BI。下文还将配合案例简单介绍Power BI的网关配置*<small>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182414113.png)

一、Microsoft Flow能够为我们做些什么？
-----

1.当一件新的产品在你的SQL SERVER数据库被创建后，新产生的数据自动加载到Power BI Service 数据集：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182426114.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

2.自动保存Outlook新邮件中的附件到你的OneDrive云盘：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182437867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

3.获取所有包含你所设置的关键词的推文以及该推文的评论并自动发送至Power BI流数据集：（假定你已经在Power BI Service创建了流数据集)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182443741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

4.更多的用法：仅仅针对于Power BI就有多达三十个左右Flow模板（你也可以自定义工作流以满足特定需求）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182449719.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*([由此](https://asia.flow.microsoft.com/zh-cn/)进入Microsoft Flow界面)*

二、如何使用Microsoft Flow提升我们Power BI的使用体验？
-----

在Microsoft Flow中，有两种流组件(连接器)：触发器和执行器，在Flow中，Power BI的触发器联结Power BI Service的数据警报功能，比如当报表的销售数据达到经设定的一个阈值时，即触发该警报；而Power BI的执行器则是接收来自触发器的数据并插入到Power BI数据集或流数据集。本文以应用最多的前者作为案例，将在下文实现当Power BI报表中的销量数据超过1万时，自动给我（或指定的其他人）发送警报邮件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182500494.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 1. 在Power BI设置数据警报
在Power BI Service的DASHBOARDS中，打开你想要设置数据警报的磁贴的菜单，选择Manage alerts,并设置好触发名称以及警报条件:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182507648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

设置完成后你可以在设置中看到警报器的运行状况，接下来将使用Microsoft Flow使该警报器能够为我们做更多的事：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182520875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 2. 在Microsoft Flow建立流程
你可以在Microsoft Flow(甚至PowerApps)上建立空白流程，然后逐步根据你的需要添加组件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182605853.png)

或者你可以类似本案例一样直接使用预设的模板，使用你的账号登录到邮箱和Power BI以创建连接：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182637783.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

在触发器中选择刚刚在Power BI新建的数据警报，然后在执行器中，指定接收数据触发警报信息的收件人(甚至可以在高级设置卡中设置抄送及附件), 同时也可以设置邮件内容，并可以使用HTML的标签来设置文本格式:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182655767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*(友情提示:设置完成后别忘了保存)*

#### 3. 测试(附网关配置)
我将在数据源更改数据，使销量大于10000，利用网关更新数据后，触发警报接收邮件：
【为使在Service上的数据刷新，需提前安装并配置网关(Gateway), 本案例使用Power BI网关个人版，相关资料见此。按如下设置：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182709379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*注：使用Power BI On-premises Gateway 个人版，你需要保证安装该网关的服务器能够读取到该数据源*

为数据集设置好网关后更新数据集，在Power BI将会收到来自警报器的通知，因为此时数据已经超过了一万：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182720327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

你在Flow中设置的收件人也将收到你所设置的邮件通知：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182728958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182738528.png)

三、其他
-----
在Microsoft Flow中，还有很多可利用的连接器，但有一些需要Premium账户才能使用。但相比用户的需求来讲，我想一定是不够用的。这个时候就有必要开发一个自定义连接器了，想一想你需要何种连接器以更好的服务于你的Power BI和工作流程吧~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129182747111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

