---
layout: post
title:  Power BI 数据安全之敏感度标签 (下)
date:   2020-07-02 06:03:50 +0000
image:  14.jpg
tags:   [Power BI,Microsoft 365]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---


*在上一篇，已讲解了敏感度标签的创建与发布，本篇讲解在Power BI Service实施敏感度标签的流程。*

### 在Admin Portal启用敏感度标签

接[上文]({{site.baseurl}}/pbi-sensitivity-label-part1/)，返回Power BI Service，在设置菜单栏打开Admin Portal页面，启用敏感度标签，你可以在此设置敏感度标签针对哪个用户组可用，或应用于整个组织：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702143624561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

完成后，系统将提示设置将在15分钟后生效。

（注：如果你此前未完成上篇的敏感度标签创建，且你的组织此前未创建任何敏感度标签，则此处的设置将不会成功）

### 为PowerBI报表和仪表板应用敏感度标签

进入你的Power BI工作区，选择你打算设置敏感度标签的报表，右键点击设置，在右边菜单栏中，如果此前的设置一切顺利，你会看到敏感度标签设置框，并且包含了你在Microsoft安全与合规中心所创建的敏感度标签（见[上篇]({{site.baseurl}}/pbi-sensitivity-label-part1/)）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/202007021443132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

现在，通过这种方式你就可以轻松为报表或仪表板设置敏感度标签了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702144619774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

此外，为PowerBI数据集设置敏感度标签，右键设置在下图处选择即可：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702145251590.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 将标签应用到Power BI报表的导出文件（.xlsx文件为例）

现在，你寄希望于用户在你的PowerBI报表导出数据后能够在Excel里看到标签，如果您的组织使用的是O365订阅版的Office，那么到此你已经完成了整个设置。但如果你们使用的是独立版本的Office，那么你还需要完成一个步骤。

你需要[安装Azure Information Protection](https://www.microsoft.com/en-us/download/details.aspx?id=53018)，这样才能把包含敏感度标签应用的插件安装在你独立版本的Office应用当中：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702150555681.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

安装完成后，打开Excel （本人使用的是2016专业增强版），如果你发现登录界面已经拥有了Azure 信息保护的LOGO，代表你安装成功：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702150755234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

现在你可以为你的任何Excel文件设置敏感度标签了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702151029677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

回到Power BI, 打开任意报表，导出数据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702110648395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

在Excel中打开它，如下图所示，标签应用成功：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702151229151.png)

不过需要说明的是，此处虽然注明了"机密", 但该标签在功能上属于"公开"性质，要设置带有数据保护功能的敏感度标签，你的组织需要拥有Microsoft Azure信息保护P1或P2的License，此外你还需要配置好Microsoft Cloud App Security控件，才能完整实施数据保护，否则，标签将仅仅只是一个"标签"。