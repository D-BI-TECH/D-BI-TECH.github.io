---
layout: post
title:  PBID外部工具DAX Beautifier使用必读
date:   2020-08-03 01:03:50 +0000
image:  06.jpg
tags:   [Power BI,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 无
---

### 关于DAX Beautifier

DAX Beautifier是由本人（Davis.Z) 开发的一款Power BI Desktop外部工具，同时也是全球Power BI社区第一个基于Python语言开发的PBID外部工具，它的作用是可以使你一键美化PBI文件中的所有DAX公式，增强代码可读性并大幅提升开发效率。

事实上，该工具早在2个月前已开发完成，但由于当时PBID尚未迎来7月的更新，还没有加入外部工具这一功能，因此当时的方案是需要手动解压PBIX或PBIT文件，然后运行程序修改文件夹下的DataModelSchema，将文件中的DAX代码替换为经过美化的DAX代码，通过这种方式来实现PBI文件全部DAX代码美化的效果（具体原理参考[《一分钟格式化所有DAX及M语句》](https://d-bi.gitee.io/dax-m-formatter-tool/)）。

现在有了外部工具，我就可以使程序直接和Analysis Services进行交互了。过去，一些第三方工具，如Tabular Eidtor对AS模型只能做到Read-Only，而现在可以支持将修改直接应用到PBID，尽管该工具使用.NET进行开发，那么理论上我使用Python也能够实现该效果（而并非像某些人说的需要权限）。尽管除了Tabular Eidtor开放的源代码之外几无资料可供参考，幸运的是，我在不到两天的时间里摸索出了具体方法。就在上周五（7月31日），DAX Beautifier正式诞生，它完美地实现了所有DAX公式（无论是计算表，计算列还是度量值）的一键美化！

### 下载与安装

下载：目前提供的是第一个版本的测试版（v1.0.0 beta) ,可[点此到达](https://github.com/DavisZHANG-BlogOnly/dax-beautifier)。后续的版本也会发布到此地址。

安装：跟随向导即可，如下。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803110303755.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)



### 使用前的注意事项

使用该工具前请确保您已满足如下要求：

1.请确认你的Power BI Desktop的版本支持外部工具（版本号应不小于：2.83.5894.661）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803110918521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

2.确认你已为PBID开启增强元数据集：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803111024197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

3.确保你的表格模型能够正常的连接到数据源（若出现类似下图情况则可能导致工具失效）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803111231330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

4.确保您报表所在的设备能够连接到互联网。由于该工具的DAX代码美化需要依赖于[daxformatter.com](https://www.daxformatter.com/)提供的API接口，您报表的每一个DAX公式的格式化都相当于发送了一次post请求，因此无法连接到互联网或中途网络断开连接都可能会导致代码美化的失败。


### 使用演示

该工具的使用可参考如下视频（已上传至[知乎](https://www.zhihu.com/zvideo/1273579251229470720)）：

<iframe width="500" height="315" allow="autoplay" src="https://player.youku.com/embed/XNDc3OTkyODE3Mg==?client_id=644f9cfcb7c5a867&amp;password=&amp;autoplay=false#cloud.youku.com" name="iframeId" id="iframeId" frameborder="0" allowfullscreen="true" scrolling="no"></iframe>

### 限制

在当前PBID版本（2020年7月），受制于计算表的兼容级别，针对于计算表内的计算列的格式化无效。

*该问题可能会在后续版本解决*

### 问答

问：为何我使用DAX Beautifier后,我的报表中的DAX并无任何变化，并且我已满足前述的所有前提条件。

答：在个别情况下，DAX Beautifier返回的格式化的结果不会立即反映到Power BI Desktop内，但此时模型内部的格式化已经完成，你可以保存报表并重启PBID进行验证。如果情况依然未解决，请刷新数据集后重试。此外，如果您的公式本身具有语法错误，那么他将保留原来的格式而不会被美化。

### 后续版本

本文基于最初版本1.0.0编写，后续版本可能有变化，请留意本项目Github页面的[更新说明](https://github.com/DavisZHANG-BlogOnly/dax-beautifier#updated)。

### 鸣谢

由Macro Russo的[DAX FORMATTER](https://www.daxformatter.com/)提供的API服务

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803114654732.png)

由王信信（知识星球：[Power BI朋友圈社区](http://powerbiquan.com/)星主）提供的赞助

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803114617795.jpg)

以及在该工具发布之前帮忙验证测试的朋友。



