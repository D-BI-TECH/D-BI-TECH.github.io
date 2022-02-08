---
layout: post
title:  Power BI 企业数据安全 (DLP)
date:   2022-02-09 19:03:50 +0000
image:  29.jpg
tags:   [Power BI,Microsoft 365]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

(接上文)

# 三、PBI数据安全之数据防护

通过在工作区指定角色配合RLS以及OLS，我们已经实现了对PBI数据内容的权限控制，但我们无法控制的是当数据从Power BI导出后，如何持续地保护数据，以及，如何对PBI数据进行监控以使得每当探测到敏感数据时，可以及时向管理员发出警报。

应对此情况的解决方案是对Power BI实施DLP (Data Lose Prevention) 。它包括以下两个阶段：

1.Microsoft 365管理员可以为Power BI创建和定义敏感度标签，不同的敏感度标签可以设置不同的权限定义，PBI各工作区的管理员可以据情况为不同的数据集，数据流以及报表设置对应敏感度标签，以实现敏感数据即使离开了 Power BI，也能得到保护。引用[MS文档](https://docs.microsoft.com/en-us/power-bi/admin/service-security-sensitivity-label-overview#introduction)的话讲：

> When labeled data leaves Power BI, either via export to Excel, PowerPoint, PDF, or .pbix files, or via other supported export scenarios such as Analyze in Excel or live connection PivotTables in Excel, Power BI automatically applies the label to the exported file and protects it according to the label's file encryption settings. This way your sensitive data can remain protected, even when it leaves Power BI.

2.Microsoft 365管理员可以为Power BI创建DLP策略，该策略可以基于PBI内容的敏感度标签对敏感数据进行监控 (该功能尚处于预览阶段)。

> 注：实施DLP需要Microsoft 365 E5 订阅

## 在PBI实施DLP

#### 创建并发布敏感度标签

1.在M365合规中心按下图创建标签。

> 注：如果Information protection选项卡不可见，则需要检查下你的账户权限以及M365 Lisence

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe4904afd34640f58ec2478915a9fd89.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

2.选择Files & emails

![在这里插入图片描述](https://img-blog.csdnimg.cn/5af6a2837d204b449ecb45434ef46136.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

3.此步骤，即是决定敏感度标签应用后的权限设置，可以添加用户，安全组甚至域，为他们分配不同的角色。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5a9957966ae428385ac8edf6ca8c3f5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

下图明确了不同角色的权限范围，也可以通过勾选具体项自定义角色的权限。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f089eefdcc9444f2bb9a860edf000d1a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

4.标签创建完成后，发布标签：

> 注：可以在已有敏感度标签创建子标签，子标签可以在继承父级的权限范围的基础上对权限进行进一步定义，这将对大型组织十分有用

![在这里插入图片描述](https://img-blog.csdnimg.cn/72f77f5414cb48b6b5ee2dc07591d952.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

在发布标签过程中，可以勾选此项，这将强制Power BI开发者在发布数据集与报表前对其应用敏感度标签，下文“PBI敏感度标签强制实施验证”部分将展示其效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f6416e2f99e8436691d3954048379ccf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

回到Power BI Service, 发布的标签已经可用：

![在这里插入图片描述](https://img-blog.csdnimg.cn/43938550ed2149689cb628ac7c7fa387.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)


> **注意：**
> - 敏感度标签发布后，通常不会立即在各个应用中显示。如下图1已经说明“It can take up to 24 hours...”，因此，如果没有在PBI看到发布的标签，并不一定代表你的设置不成功，只需耐心等待。
> - Power BI管理员需要在租户设置中启用敏感度标签（见图2）。

![图1](https://img-blog.csdnimg.cn/5b30a66fe4e34053b9220627f7dc5d1b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

![图2](https://img-blog.csdnimg.cn/a26ef59258664f72b1d83ca4eaec22c1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)


#### 验证

1. PBI敏感度标签强制实施验证。

	如下用户必须先为PBIX应用敏感度标签，才可发布报表。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a601224cd874434fa119d85516f70375.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

2. 数据导出保护效果验证

	如图，我已为数据集，报表以及分页报表指定了敏感度标签，其中HIGH SENSITIVE只有特定用户才能查看导出的数据。

![在这里插入图片描述](https://img-blog.csdnimg.cn/0160bed5bba948f0bf21ad03b1f65127.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

【示例1-1】Power BI报表导出验证（使用有权限的账户）

![在这里插入图片描述](https://img-blog.csdnimg.cn/e24b1194735d4cc3ac8a998d871a4a0e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

【示例1-2】Power BI报表导出验证（使用无权限的账户）

![在这里插入图片描述](https://img-blog.csdnimg.cn/183444e8d6254cf39e7fefe793d88b2d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

【示例2】Power BI分页报表（Paginated Report）导出验证

![在这里插入图片描述](https://img-blog.csdnimg.cn/765c9eb7e2b0492387ff5111792180c7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)


#### 创建DLP策略

DLP策略在Power BI尚为预览功能，部分特性暂时不支持，如果公测在今年四月，那么GA可能又要往后推，下文仅就策略的创建部分做部分说明。

![aaa](https://img-blog.csdnimg.cn/ec99a4ee3f0a443b989b22dc8b002728.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

1. 在M365合规中心，从Custom Policy开始创建：

![在这里插入图片描述](https://img-blog.csdnimg.cn/44b79b92e90f4b468f16dfb62c9b0989.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

2. 选择Power BI，DLP默认应用到整个容量。如果仅希望DLP应用在个别工作区，可以输入对应的WorkspaceID:

![在这里插入图片描述](https://img-blog.csdnimg.cn/75033d2569b6452db233fda4ae3c7c0b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

3. 创建DLP策略警报触发规则，可以仅就标记了某特定敏感度标签的PBI内容实施：

![在这里插入图片描述](https://img-blog.csdnimg.cn/cdb97b6f35c844339f1a97da63a5c0d8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

4. 管理员可在此处进行数据监控：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e44cf4f830ec4e02a9ebf04c494279d7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

## 其他

除上述内容外，有关Power BI安全性的技术点还有很多，如网关安全性，私有链接集成，Azure Log Analytics集成等等，要逐一讲清这些内容，足够写完一本书了，而国内外关于该领域的研究非常少，一些坑，一些bug，即使搜遍谷歌也难以有对应解决办法，还是要依赖于读者的耐心探索，并在必要时获取微软技术人员的支持。

## [附]相关链接
   
 -  [Power BI security white paper](https://docs.microsoft.com/en-us/power-bi/guidance/whitepaper-powerbi-security).



-----------------

**关注作者： [知乎](https://www.zhihu.com/people/zhang-zhe-hong-01/posts)   [Power BI官方社区](https://community.powerbi.com/t5/user/viewprofilepage/user-id/220984)**
