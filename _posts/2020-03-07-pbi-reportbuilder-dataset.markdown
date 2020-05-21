---
layout: post
title:  PBI Report Builder 系列：配置数据集
date:   2020-03-07 08:03:50 +0000
image:  01.jpg
tags:   [Power BI,分页报表]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---


*前述：本文作为分页报表文章系列的一部分，面向零基础用户演示在Power BI Report Builder连接数据源及配置数据集的方法及相关知识。*

### 新的开始

首先，打开下图左下角的Power BI Report Builder, 而非右上角的Report Builder (该版本只能本地部署）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200307200929415.PNG)

（附：[PBI Report Builder 下载链接](https://www.microsoft.com/en-us/download/details.aspx?id=58158)）

### 关于数据源

打开Power BI Report Builder后，我们第一步是连接数据源，然后在该数据源之下配置数据集。根据[官方文档](https://docs.microsoft.com/zh-cn/power-bi/paginated-reports/paginated-reports-data-sources)所述，分页报表支持的数据源分两类，一类为可以在Power BI Services直接受支持的数据源，如下：

- Azure SQL Database （位于云端）
- Azure SQL Data Warehouse（位于云端）
- Azure SQL Managed Instance（位于云端）
- Azure Analysis Services（位于云端）
- Power BI dataset（位于云端）
- Premium Power BI dataset (XMLA)（位于云端）
- Enter Data（存储在报表内部）

另一类为本地部署的数据源，通过Power BI Gateway来实现对数据的安全访问，如下：

- SQL Server （位于本地）
- SQL Server Analysis Services（位于本地）
- Oracle（位于本地）
- Teradata（位于本地）

下文将以大家最常用的Power BI数据集为例，讲述连接数据源并配置数据集。

### 配置数据集 -- Power BI Dataset 为例

如果你已经在Power BI Services发布过Power BI报表，那么你可以通过下图方式连接该报表的数据集，首次操作会提示登录。

![demo](https://img-blog.csdnimg.cn/20200307204313522.PNG)

出现下图界面表明连接成功，你可以在你的Power BI工作区中任选一个数据集作为测试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200307204929798.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

如果你在Power BI Services没有任何数据集，你也可以连接本地Power BI数据集，有经验的用户或许已经想到了DAX Studio这个强大工具，没错，通过DAX Studio可以使你知道本地Power BI数据集的端口号，这样你就可以通过该地址，使用SSAS的方式连接到本地Power BI数据集：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200307220954153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

本文选择PowerBI Blog Union作为演示，该数据集即"[Power BI中文博客联盟]({{site.baseurl}}/pbi-cn-blog-union/)”这个报表所使用的数据集，该数据集是利用PowerQuery获取来自知乎及网站不同作者的文章数据。接下来配置数据集：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030720542890.PNG)

你会进入到以下界面，我用不同颜色的方框作为标识来讲解：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200307210640252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

- 底部的灰色方框用于设置连接超时限制，此处通常保持默认值。

- 黄色方框"Import"是允许你导入其他分页报表的查询语句（也可以导入.sql文件，如果你连接数据库的话）,这样对于同样的数据集无需麻烦你每次重写。

- 蓝色方框是刷新字段，当你修改了原来的数据集后，比如新增一个字段，你需要手动点击此处刷新，它不像PowerQuery那样可以在你每次修改查询语句后为你自动刷新。

- 绿色方框允许你在查询语句中使用函数，它可以实现十分复杂的功能。它可以根据你所设置的条件来改变或切换数据集，结合参数，它可以实现允许前端的报表用户在查看报表前，使用哪个数据集（类似的，也可以实现让用户在前端选择使用哪一个数据源，结合参数来改变报表的连接字符串），此处属于进阶内容，此处点到即止。
- 最后红框的”Query Designer"即为我们要选择的项，它允许你用拖拽的方式构建你的数据集，对入门用户十分友好。

选择”Query Designer"后，进入下图界面：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200307214100848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

无论你选择使用DAX还是MDX作为查询语言，都无需担心，因为你可以通过拖拽的方式在右边空白处构建数据集，系统为为你自动生成查询语句。但由于连接的是Power BI数据集，本质上是SSAS表格模型，因此使用DAX作为查询语句性能更好。构建数据集的方式是，在左侧选择必要的维度，然后在"Measures“中选择度量值作为数值，这里的Measures都是你在Power BI Desktop中设置的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020030721523066.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

拖拽后点击上方感叹号执行查询，那么你会看到右侧的数据，它显示了知乎多位作者不同领域的文章计数，你还可以在上图红框处设置过滤，比如排除空值。点击OK后，系统自动为你生成如下语句:

```SQL
EVALUATE
SUMMARIZECOLUMNS (
    '标签'[标签1],
    FILTER ( VALUES ( 'ZhiHuPost'[标签1] ), ( 'ZhiHuPost'[标签1] <> "" ) ),
    "post_count", [post_count]
)

```

由于该查询较为简单，因此你完全可以自己写DAX查询，但是当查询复杂时，比如需要利用参数，多选以及更多表关联时，你会切身体会到Query Designer的便利。

至此，数据集已配置完成，你可以在设计报表时直接使用这些字段了，但请注意，数据并未导入报表，这和Power BI不同，如在序篇中所述:"只有用户在使用报表的时才会执行查询",我们在此只是定义了一种关系，报表设计在字段定义层面进行，而不需要依赖数据本身。这使得分页报表文件(.rdl)即使用于大数据项目依然能保持本身的轻量化，如果是在Power BI，意味着你需要把上亿行的数据先导入再进行处理，极大降低了报表开发的效率，当然，除非你愿意使用问题更多的DirectQuery。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200307221516231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)


### 结语

对于其他类型的数据源，配置过程大同小异，比如连接本地数据源SQL Server，区别是需要写SQL语句或使用现有存储过程等方式配置数据集（可参考本文中的截图）。接下来将是分页报表开发最核心的部分：报表设计（与可视化）。




#### 【附】系列目录
 - [x] [序篇]({{site.baseurl}}/pbi-reportbuilder-2/)

 - [x] 导入（配置）数据集 （本文）

 - [ ] 报表设计

 - [ ] 报表发布
