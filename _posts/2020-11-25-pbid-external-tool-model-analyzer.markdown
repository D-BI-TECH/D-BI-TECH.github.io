---
layout: post
title:  PBID外部工具：Model Analyzer
date:   2020-11-25 01:03:50 +0000
image:  20.jpg
tags:   [Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 无
---

### 前述

上个月PowerBI.Tips社区负责人希望我开发一个PowerBI模型Document工具，用于管理和分析PowerBI表格模型，度量值，表关系等，当时我对此兴趣不大，因为这属于冷门需求，而且针对于模型主要的性能分析也可以在DAX Studio中完成，但当我看了Meagan Longoria的博客[Documenting your Tabular or Power BI Model](https://datasavvy.me/2016/10/04/documenting-your-tabular-or-power-bi-model/)后，意识到该需求还是有一定必要性，该文里讲述了利用DMV查询来获取表格模型的全部信息,  并且提供了PBIT文件，你只需要输入表格模型实例名称以及数据库名称即可建立Power BI模型分析，这是个不错的想法，因为利用PBI既有的交互能力以及动态的数据刷新来分析模型再好不过了，因此我打算借鉴和优化该PBIT，然后把它改造成一个PBID外部工具，使其运行更快捷便利。

### 关于 Model Analyzer

Model Analyzer全称Power BI Model Analyzer, 即Power BI模型分析器，你只需运行安装程序，然后从Power BI Desktop外部工具栏运行它，即可打开一个专用的PBIT文件，在Meagan Longoria提供的旧版本中，你需要自行查询实例和数据库名称（通过DAX Studio或其他方式）并手动输入它们，但此处，该PBIT会自己获取这些参数值，你只需选择对应的实例名以及数据库名，点击加载即可。Power BI会运行多个DMV查询，包括模型，表格，列，KPI以及度量值等数据，一些非常用的模块，如透视，计算组等模块暂不包括在内，以后可能会被包含在高级版本中。

### 下载与安装

[点此](https://github.com/D-BI-TECH/Model-Analyzer)到达Github页面，下载Power BI Model Analyzer.zip文件解压并运行即可一键安装。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125161934414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

*未来如果发布稳定的版本，则可能会发布在[Business Ops](https://powerbi.tips/product/business-ops-beta/)平台供广大海外用户安装，我会和PowerBI.Tips对其做进一步评估*

### 使用前注意事项

就目前版本（1.0.0 beta）而言，用户需要在Power BI Desktop进行以下设置【始终忽略隐私级别设置】：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125163618551.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

否则会在加载数据时出现以下错误：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125163748623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

此外，暂无法连接使用PBID-RS版本制作的表格模型。

### 图文演示

如同[DAX Beautifier](https://d-bi.gitee.io/pbi-external-tool-dax-beautifier/)，使用该工具极为简单。安装后，打开你需要分析的PBIX报表文件，然后再外部工具栏点击图表运行即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125164504274.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

运行PBI Model Analyzer后，会打开一个PBIT文件，并弹出如下窗体，该文件会自动获取最近一次修改过的（通常也是正在运行的）PBIX文件内的表格模型的实例名称以及数据库名称（如前文所述），这大概需要10秒左右的加载时间。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125164732358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

在下拉框中选择自动获取的参数值即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125165403399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

首次连接，需运行多次查询：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125165916297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

加载完成后，你就可以查看并分析您PBI报表所有模型信息了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201125171923558.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_left)

如果你想将模型信息导出到Excel或数据库进行进一步分析，则可以使用DAX Studio连接此模型并使用数据导出功能。


### 后续版本

该工具还有很大的优化空间，功能也可以有很多拓展，目前测试版本的模型分析是以【Analyst in Power BI】形式运行，后续会考虑加入将模型信息一键导出到Excel等其他功能，对我而言实现这些功能并不难，只是时间的问题， 主要是该工具的受众群体的确较小，也许在所有PBI开发者中仅不到10%的人会需要，但这些人当中，尤其是我在PBI国际社区认识的群体里，对模型记录的需求却又很频繁，这也是我尝试开发该工具的原因，但后续该工具是否继续优化，还是未知数，这完全取决于社区的需求。
