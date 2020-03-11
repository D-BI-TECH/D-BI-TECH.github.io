---
layout: post
title:  PBI Report Builder 系列：报表设计
date:   2020-03-10 08:03:50 +0000
image:  02.jpg
tags:   [Power BI,分页报表,可视化]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---


本文主要讲述分页报表的设计方法，并默认您已阅读过上一篇文章（PBI Report Builder 系列 数据集配置）或已经掌握在Report Builder连接数据源及配置数据集。

### 内容概览

本文以利用Power BI Report Builder 构建三大财务报表之一的资产负债表为例，讲解以下知识点：

- 构建表格/矩阵
- 图表简介
- 效果预览
- 单元格及字体格式控制
- 无处不在的Fx
- 表格过滤与排序
- 参数的使用

下图是我为本文制作的资产负债表样本，其结构简单，适合本文定位，且作为财务专用的正式报表能够一定程度上体现分页报表的优势。如果你认真阅读本文并结合亲自实践掌握以上知识点，那么你也能很好的完成类似的报表（或做得更好）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200310232223348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 数据集介绍

本文使用微软提供的最常见的数据库[AdventureWorksDW(点击下载)](https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2017.bak)作为数据源, 取其中相关表构建资产负债表的数据集，其前10行如下示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200310233735475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

其中，ReportType用于区分数据是属于年报(Annuall)还是月报(Monthly)，IsBalance用于区分对应的科目是否会被用于资产负债表，如果为0代表不会，1代表会。比如第一行数据的科目类型属于“Expenditures"，即支出，该科目主要在利润表或损益表中使用，不会被用于资产负债表。这两列是我在取数时在SQL语句中自己加上去的，源数据中不存在这两个字段。你可以自行构建数据集，只要可以满足你制作资产负债表即可，无需完全和本文一致。

### 构建表格(或矩阵)

Report Builder 提供两种构建表格的方法，手工构建与使用向导。其中手工构建法即点击下图"Insert Table"，这使你可以在画板上新建一个两行三列的空白表，后期你需要根据自己需求添加行或列，以及填放数据，设定结构，这些任务中，大多数都可以使用向导完成，可以省去繁琐的操作，现点击下图”Table Wizard"进入向导：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200310235100712.PNG)

在这里你只需简单的拖拽（类似数据透视表）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311092704731.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

完成后点击下一步，提供了几个关于报表的选项，包括“是否显示总计”，并提供三种布局方式，右侧会有效果预览，本文选用第三种"Stepped,Subtotal Above", 并勾选Expand/Collapse groups, 该选项允许用户对报表不同层级进行展开或折叠，如下图中点击加号后会展开子层级科目的数据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311000524543.png)

完成后继续点击下一步，会提供最终预览，点确定后表格构建初步完成。

### 图表简介

构建图表与构建表格或矩阵原理相同，也提供对应的向导，方法大同小异，在此不必细说。以下是Power BI Report Builder 提供的默认图表，相对而言较为丰富，你可以根据你的需求选择。这里相比Power BI 报表的其中一个优势是任何图表都可以被嵌入表格或矩阵中的单元格内，制成Mini图；你还可以把你已经设计好的图表发布到Power BI Report Server 进行组织内部的共享。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311001045857.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 效果预览

你可以在画布中初步构建好报表后，点击【Home】-【Run】- 【View Report】运行报表预览效果，下图是构建好（已完成表格过滤与排序步骤，详情参考下文）但未进行任何格式美化的报表截图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311002412527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 单元格及字体格式控制

为了对报表进行美化，需要调整报表格式，这通常是右键点击需要修改的单元格，在弹出菜单点击"Text Box Properties",会弹出下图界面。这里包含了十分丰富的格式调整选项，除下图展示内容外，在其他选项卡还包括数据显示格式，显示隐藏控制，交互式排序以及最强大的"Action"选项卡中提供的下钻到子报表等高级功能。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311002852216.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

除了通过点击"Text Box Properties"进行格式调整外，还可以利用属性窗口进行调整，特别是你需要一次性修改多个单元格的格式时，利用属性窗口可以一次性批量完成修改，无需逐次修改每个单元格。属性窗口通过勾选"Properties"开启。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031100340053.png)

### 无处不在的Fx

在Report Builder，就如上图界面所示，你可以看到Fx标志已经覆盖了报表设置的方方面面，这意味着公式的重要性。利用分页报表的公式，你可以让报表变得十分智能。由于在分页报表，报表设计是根据Row Group以及Column Group, 并在格式与功能上精确到单元格，这使得你能够结合公式进行控制，设计出任何能够在Excel中完成却无法在Power BI完成的复杂格式报表。在本例中，以一个简单的例子展示其冰山一角：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311004419414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

上图中报表的副标题"Annual Report"不是固定不变的，它会根据你所选的报表日期而变化（关于筛选部分在下文说明）。运行报表后，如果你所选的日期是该年度12月最后一个数据核算日，那么此处显示“Annual Report”，同时鼠标悬浮会有中文提示"年报"，否则，副标题显示"Monthly“，鼠标悬浮也会改为"月报"。实现这一简单有实用的功能就是通过公式进行控制，比如鼠标悬浮提示的部分公式如下：

```SQL
=IIF(
    First(Fields!ReportType.Value, "AdventureWorksDW_Finance")="Annual",
"年报","月报")
```

其中，”AdventureWorksDW_Finance“是数据集的名称，代表该公司是根据此数据集的ReportType字段进行条件判断。由此我们就实现了智能表头控制，此外，如你所见，利用公式还可以控制单元格及字体格式，行列隐藏显示规则甚至修改数据集等等。下面就介绍利用公式的另一个常用用例。

(关于Report Builder全部函数使用说明，参考[此文档](https://docs.microsoft.com/en-us/sql/reporting-services/report-design/expression-reference-report-builder-and-ssrs?view=sql-server-ver15)）

### 表格过滤与排序

在画布下方Row Groups处选择父级组（AccountType），点击Group Properties进入下图界面：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311005748382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

如果你没有进行表格过滤，那么运行报表后的结果将显示全部科目的数据，对于打算设计纵向布局的资产负债表那么此处不必改动，而本文示例的效果是左右布局，资产科目在左侧，负债和所有者权益科目在右侧，因此我们需要在左边和右边各放置一张结构相同的表，然后使用过滤规则让左右两边都只显示其对应的AccountType的科目。参照上图及下图分别对左右两表设置过滤：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311093312875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

同样，你可以点击Fx，利用公式创建表达式，对表格进行更加定制化的行集过滤，但如果你不仅需要过滤特定行，还需要过滤特定列，你只需在画布右下角的Column Groups处进入Group Properties进行对应设定即可。接下来点击“Sorting”选项卡，此处用于表格排序，界面如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020031109461134.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这里类似于Power BI的Sort By Column的功能，你可以让你的报表依据数据集某一字段进行降序或升序。你也可以点击“Add"设置次要排序规则，就像在Excel中那样。但这里的强大之处同样是允许你利用表达式控制排序规则，这意味着你可以让你的报表在满足某些条件时按A列优先排序，在其他情况按B列或其他列优先排序，规则由你在表达式中依据需求自由设定。

### 参数的使用

在Power BI Report Builder, 你可以通过设置参数来影响报表生成结果，关于此，我总结有如下三种用法：

1. 将参数作为数据集过滤条件，起到切片器的作用（最常用）
2. 将参数作为输入框，在报表中输出
3. 将参数作为开关，影响报表展示的行为

##### 第一种：用作切片器

用参数过滤数据集，需要在数据集中配置变量。这里针对两种主流数据源进行说明，对于数据源为数据库的，通常是使用SQL语句构建数据集，那么只需要在WHERE语句中直接引用变量即可，无需声明；对于存储过程则在存储过程中声明变量，然后在查询语句中引用变量。对于以Power BI Dataset或SSAS表格模型作为数据集的，使用的是DAX构建数据集，在Query Designer按下图操作，设置过滤条件后勾选Parameters:

(关于Query Designer参考上一章--[配置数据集]({{site.baseurl}}/pbi-reportbuilder-dataset/)）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311100820313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

配置完成后会自动生成以下DAX查询语句。此处有一个也许你从未用过的DAX函数RSCustomDaxFilter，该函数实际上是专用于Report Builder的DAX函数（即使在SSAS 2019也没有），其实对于以下查询，你完全可以用Filter代替，但该函数的意义何在？其作用在于处理多选。如果你有一个数据集，包含100家店铺，你需要从中选择十家店铺，在Power BI, 你不需要Filter函数，因为它有筛选上下文机制可以轻松处理这种多选，然而在Report Builder中却做不到Power BI那么简单，对应的操作都需要DAX函数完成，故而此情况下DAX查询会随着多选项数的增加而愈发复杂，在Chris Webb的[此篇博客](https://blog.crossjoin.co.uk/2019/11/03/power-bi-report-builder-and-rscustomdaxfilter/)中也提到了DAX查询的长度限制问题以及可能导致的报错，因此RSCustomDaxFilter函数就是专门用于应对Report Builder中的这种特殊情况而设计的。

```
EVALUATE 
SUMMARIZECOLUMNS('标签'[标签1], 
    RSCustomDaxFilter(@标签标签,EqualToCondition,[标签].[标签1],String), 
    "post_count", [post_count])
```

数据集变量设置好后，此时回到主界面，参数会为你自动生成：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311101410482.png)

运行报表后你就可以使用参数过滤数据了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311103506366.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

回到我们资产负债表的项目，这里的筛选机制是单选。右键参数选择属性，你可以设置参数的默认值，可用值以及是否允许空值，是否允许多选等等：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311103913248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

顺便一提，在Select parameter visibility处，你可以选择Internal或Hidden隐藏参数。这两者的区别是，选择Hidden仅仅是隐藏了切片器，但用户依然可以通过在报表链接添加参数过滤报表，但Internal则是完全内置了参数，相当于对用户禁用了该参数的使用。

##### 第二种：用作输入框

该用法允许用户在输入框中输入文字或数字，然后将结果直接打印到我们的报表。在本文案例的资产负债表中，我设置了Signature参数，财务负责人在审核报表无误后需要签署，在Signature输入框中输入姓名，如此处"Davis ZHANG", 运行报表后，所输入的名字就会被打印到报表中：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311105058774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

该用法还有很多用途，比如你是一名数据分析师，你需要指定一个报表模板，以便每月总结时使用，但每次的数据不同，分析就会不一样，通过设置输入框就允许你将你对数据的见解，写好粘贴到输入框中，内容就可以直接打印到报表你所设的区域内（然后根据需求你可以将报告导出成多种格式，Excel,PPT,PDF等等）。

##### 第三种：用作开关

该用法同样用途广泛，比如最简单的报表颜色，在公式中引用参数，你可以允许用户切换报表的颜色主题（黑纸白字的夜间主题还是白纸黑字的日间主题），或者是否隐藏某些数据等等等等，只有你想不到的，多数功能都可以完美实现。本文案例中则是一个比较概念性的功能--盖章动作，选择True则会在报表中盖章以供后续打印：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311110530535.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

此处是设置也很简单，这个章其实就是个图片，用户通过参数来控制是否显示它，你可以自行探索，制出满足您需求的功能更强大的报表。

### 总结

到此，如果你结合本文亲自实践，相信你已掌握使用Report Builder设计报表的技能，举一反三，灵活运用。报表做好后你还可以尝试使用导出功能，体验到Power BI Report Builder**所见即所得**的优势。下方截图来自Excel：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200311111539993.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)



#### 【附】系列目录
 - [x] [序篇]({{site.baseurl}}/pbi-reportbuilder-series/)

 - [x] [导入（配置）数据集]({{site.baseurl}}/pbi-reportbuilder-dataset/)

 - [x] 报表设计 （本文）

 - [ ] 报表发布
