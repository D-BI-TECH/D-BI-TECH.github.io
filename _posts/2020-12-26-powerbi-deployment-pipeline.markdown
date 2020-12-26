---
layout: post
title:  Power BI 部署管道（CI/CD) 完全攻略
date:   2020-12-26 01:03:50 +0000
image:  25.jpg
tags:   [Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*本文讲解Power BI 部署管道 （deployment pipeline）基本使用方法及其完整特性*

### 何为Power BI 部署管道

在讲解Power BI部署管道之前，先简单了解一下什么是CI/CD。CI/CD全称译为“持续集成（CI)，持续分发和持续部署(CD)”，谷歌，百度也有不少关于此的解释，但简单讲，它就是一套方法论，涵盖产品从开发，到测试再到发布的一整套流程。对于BI而言，就是将这套流程应用到报表开发上，从用户提出需求最后到报表新版本的发布，形成一个稳定且持续的闭环（如下图所示），使得报表发布质量得到保证的同时，也能大幅提高开发效率。

![报表开发持续集成部署](https://img-blog.csdnimg.cn/20201226163626278.png)

Power BI部署管道（deployment pipeline）即是微软针对Power BI报表, 数据集以及仪表板推出的一个准CI/CD工具，它完整地提供了实现该流程所必须的三个环境，即开发环境，测试环境以及生产环境。一个典型的流程是，报表开发者可以将未完成的新项目发布到开发环境，当完成后发布到测试环境，测试团队可以进行数据校验与功能测试，并与开发者反复沟通核对，直到数据集的数据足够可靠，报表的功能足以满足需求，仪表板的设计足够合理，再最终确定发布，以实现CI/CD带给我们的高质量与高效率。

![图片来自微软文档](https://img-blog.csdnimg.cn/2020122617112396.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

*注：实现Power BI CI/CD的另一种思路是使用Azure Devops，参考[此文](https://adatis.co.uk/power-bi-ci-cd-with-devops-pipelines/)*

### 如何开始

首先，Power BI 部署管道目前属于Power BI Premium功能，需要你满足以下条件中的任意一种：

1. 您所在的组织已拥有Power BI Premium订阅，且你具备相关权限。
2. 您拥有Power BI Premium Per User (PPU) 订阅 

*如果你想了解Power BI PPU，或者你不具备以上条件，仅仅拥有Pro License甚至免费账户，可参考：[什么是Power BI PPU以及如何免费试用60天](https://d-bi.gitee.io/about-pbi-premium-per-user/)*

满足条件的，在你的Power BI Service界面就会拥有部署管道选项卡，此处，单击创建管道并为其命名即可开始：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226173300636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)


### 基本流程

管道建立的基本流程很简单，首先你需要拥有一个已启用Premium的工作区，然后你可以把它分配到三个环境中的任意一个，通常而言，我们会首先分配到开发环境：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226174416392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

分配后，我们在该工作区的所有内容的数量可以得到显示，你可以任意勾选需要的内容发布到测试环境，第一次部署，系统会创建一个新的工作区，用以存放发布后的内容：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226174723915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

你还可以进一步地发布到生产环境，同样，用于生产环境的工作区也将被创建，这样，Power BI的所有内容都可以从开发环境一致复制到生产环境，一旦你将新的报表发布到开发环境，部署管道也能检测到哪些内容被修改，点击Compare即可观察：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226175315230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

*注：发布到生产环境的内容还可以向前部署，前提是需要上一个生产环境须为空*


### 权限控制

具备足够权限的用户可以基于整个管道的层级进行权限控制，还可以为每个生产环境进行权限控制，你可以为不同的用户分配不同的角色，但每个环境至少需要一名管理员。关于角色权限参考如下表格，其他详情可参考[此文档](https://docs.microsoft.com/en-us/power-bi/collaborate-share/service-new-workspaces#roles-in-the-new-workspaces)：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226180530230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

此处做一个示例。比如，你希望让所有开发者，只能将报表发布到开发环境或测试环境，那么你可以在生产环境中移除该用户或将其设为浏览者：


![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226180706844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

这样，开发者在PBID发布报表时便不能直接发布到生产环境：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226181340697.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

对于被移除的用户而言，也不能浏览对应环境的信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226181434702.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 定义数据集规则

你可以为管道中不同的环境设定不同的数据集规则。假设有这样一个场景，出于安全因素或数据敏感性等原因，管理者希望开发者在开发报表时，使用专用的测试数据集，而当数据集发布到下一个环境后，自动切换使用正式的数据集；另一个场景是，实际数据集过大（几百万甚至几千万行），数据每次导入到PBID，以及发布的过程都需要等待很长时间，因此开发者希望在开发过程中提高效率，仅仅取数据集中的前100行（比如），当发布到生产环境后，再自动切换到完整的数据集，为实现这种效果，可以点击环境右上角并选择需要设定规则的数据集：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226182651969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

规则有两种：数据集规则，以及参数规则：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226182834405.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

##### 数据源规则

其中数据源规则尤其适用于我所列举的第一个场景，当Power BI数据集发布到下个环境后，自动切换到新的数据源。但此处主要有两个限制：

1. 当前数据源和目标数据源必须是相同类型。比如使用Odata作为数据源的，也必须，且只能使用Odata数据源作为目标数据源。
2. 仅支持以下类型的数据源，此处参考官方文档：
![3.](https://img-blog.csdnimg.cn/20201226183231704.png)
 
设定的方法也很简单，直接选就可以了：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226183440717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

*但需要注意的是，确保你需要的数据集发布到管道内的工作区，否则此处不会列出他们。*

##### 参数规则

使用该规则的一个好处是不受数据源类型限制，且尤其适用与前文所述的第二种场景。你需要首先在报表的Power Query编辑器里创建好参数。比如我设定了一个叫TOP的参数，该参数默认值是"TOP 100", 我在测试环境为数据集设定了该参数，然后在生产环境取消该规则，这样就实现了报表在开发以及测试时仅取前100行数据，而发布到生产环境后则取完整数据集的效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226184142639.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

建议报表发布之前，在此处验证参数是否有效：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226184616748.png)


### 其他

你还可利用Onedrive实现PBIX文件版本控制，可参考[此文档](https://docs.microsoft.com/zh-cn/power-bi/collaborate-share/service-connect-to-files-in-app-workspace-onedrive-for-business)。任何补充欢迎留言。

-----------------

**关注我： [知乎](https://www.zhihu.com/people/zhang-zhe-hong-01/posts)  [Power BI官方社区](https://community.powerbi.com/t5/user/viewprofilepage/user-id/220984)**