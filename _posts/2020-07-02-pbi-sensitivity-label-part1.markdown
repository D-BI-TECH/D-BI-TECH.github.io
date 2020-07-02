---
layout: post
title:  Power BI 数据安全之敏感度标签 (上)
date:   2020-07-02 03:03:50 +0000
image:  13.jpg
tags:   [Power BI,Microsoft 365]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---


*上个月，微软官方宣布Power BI敏感度标签已“Generally Available”（参见[此文](https://powerbi.microsoft.com/en-us/blog/announcing-power-bi-data-protection-ga-and-introducing-new-capabilities/))，那么什么是敏感度标签？对于使用Power BI的企业或组织而言有何作用以及如何实现？官方文档的资料较为分散，本文将会在讲解概念后展示一套连贯的设置流程，给出答案。*

### 何谓敏感度标签，有何作用

Power BI Service上的敏感度标签可为你的数据集，数据流，仪表板以及报表进行标记和分类，标签的名称是自定义的，比如"个人"，"公共"或者"机密", 该标签是在Microsoft安全中心创建并定义的，因此你在Power BI使用的敏感度标签与整个Microsoft 365高度统一，这使组织对于数据文件的管理更加便利和规范。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702111928336.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

此外，敏感度标签不仅仅是一个标签，事实上它可以实现对数据的保护。不过，标签与Power BI报表或数据集本身的权限设置无关，因为其权限是由管理员在Power BI Service配置或者对特定报表实施RLS来实现的，敏感度标签不会对这些权限设置产生任何影响，实际上，它所能控制的是从Power BI导出的文件的权限，比如Excel文件(xlsx)和PPT文件,  可以为带有"敏感"标签的文件设置保护，防止特定用户修改它们，或避免他们从Power BI将文件下载到非托管设备。至于标签策略中权限的指派，则不在Power BI的设置范畴，需要管理员在Microsoft安全中心统一设置并发布。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702110648395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

接下来本文将会分为两大部分讲解敏感度标签的实施步骤，第一部分是创建和定义标签（即本文），第二部分是在Power BI实施标签（见[下篇]({{site.baseurl}}/pbi-sensitivity-label-part2/)），此处你需要确认两点，否则设置将不会成功：

1. 你所在的组织已订阅Office 365
2. 至少拥有一个Power BI Pro账户

### 在Microsoft安全与合规中心创建标签

提示：要完成次部分，需要确认你是否有足够的权限。在Microsoft安全与合规中心创建标签需要你拥有全域管理员身份，或者由你所在的组织的全域管理员在Microsoft安全与合规中心（或者在PowerShell使用命令）为你的账号分配必要的权限。

首先，打开[Microsoft安全与合规中心](https://protection.office.com/)，点击图示部分创建敏感度标签：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702114109314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

随后，跟随向导，为标签命名以及添加描述等等：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702114239119.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

接下来需要注意一点，在"Encryption"这个步骤，选择"Apply", 就是用于为你所设的标签指派数据保护功能的（此外还需要设置Cloud App Security，此设置不在本文范畴，会另行说明）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702114322647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

然而不幸的是，仅仅拥有O365的订阅是不够的，要实现此步骤带来的数据保护功能还需要单独[购买Microsoft Azure信息保护P1或P2](https://azure.microsoft.com/zh-cn/pricing/details/information-protection/), 或者订阅了Microsoft 365 商业版（包括信息保护P1) 或合规版（包括信息保护P2) 。对于不具备这种条件的，只能设置象征意义的敏感度标签，按一下图示选择"None", 这将创建一个不带数据保护能力的标签，但它依然可以随附在文件之中为组织管理文件提供便利：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702115038198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

继续跟随向导，设置完成后如下示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702134111487.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 在Microsoft安全与合规中心发布标签

创建标签后，你的组织还无法使用它，你必须发布创建的敏感度标签，才能在其他应用（如Power BI）使用它。勾选你所创建的敏感度标签，点击发布：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702135217870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

跟随向导，此过程中你可以设置将敏感度标签应用于特定的用户或者用户组，如果不设置则默认应用于整个组织，你还可以在"Policy setting"处设置文档的默认标签，如果不设置则默认不设标签，最后你需要为策略命名及添加描述。完成后，将如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702135326132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 后续步骤

随后，你就可以返回Power BI Service,在Admin Portal开启敏感度标签功能并应用了，这将在下一篇进行具体说明。