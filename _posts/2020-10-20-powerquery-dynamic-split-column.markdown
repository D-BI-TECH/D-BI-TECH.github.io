---
layout: post
title:  PowerQuery应用：动态分列
date:   2020-10-20 01:03:50 +0000
image:  17.jpg
tags:   [Power Query,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*今天在Power BI Community收到一个问题，其需求是根据产品列不同的的值来自动分列（下文提供图解），我很快想到只需定义个PowerQuery函数就可以解决，并且在此将这个小技巧分享给大家。*

### 需求

如下图所示，根据ID列（产品号）对NUMBER列进行划分，由图可知，这并非透视，而且要求每当ID列增加新的产品号时，可以自动追加新的列，以此类推。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019110456613.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)


### 数据

打开PowerQuery编辑器，模拟一份示例数据：

```SQL
let
DATA = 
    Table.FromRecords(
        {
            [ID = "A", Number = 12],
            [ID = "A", Number = 5],
            [ID = "A", Number = 6],
            [ID = "A", Number = 8],
            [ID = "B", Number = 14],
            [ID = "B", Number = 9],
            [ID = "B", Number = 7],
            [ID = "B", Number = 7],
            [ID = "C", Number = 5],
            [ID = "C", Number = 16],
            [ID = "C", Number = 18]
        },
        type table [ID = nullable text, Number = nullable number]
    )
in
    DATA
```

如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019111606480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)


### 方案

整体的思路就是使用M语句定义一个函数，ID作为函数的唯一参数。每当传参后，比如当ID为A时，原数据就可以被此参数过滤为仅有A产品的数据表，然后只需将NUMBER列重命名为参数即可，当参数为B,C,D,E...时，以此类推，因此新的NUMBER列名也会是B,C,D,E...,PowerQuery识别到这是不同的列，在合并数据时就会自动创建新的列，这样就完美解决了问题。

### 步骤

##### 定义函数

此处只需定义一个参数，完成两个步骤即可，按前文所述，过滤并重命名：

```
(ID_Code as text) => 
let
DATA = 
    DATA,
    DATA_Filtered = Table.SelectRows(DATA, each ([ID] = ID_Code)),
    DATA_Renamed = Table.RenameColumns(DATA_Filtered,{{"Number", ID_Code}})

in
    DATA_Renamed
```

代入任意参数测试无误：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019113710298.png#pic_center)

##### 新建参数表

我们需要让参数连续向函数传参，以便得到一份完整的表，因此需要取ID列并去重，以生成参数表：

```
let
    Source = DATA,
    #"Removed Columns" = Table.RemoveColumns(Source,{"Number"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Columns"),
    #"Invoked Custom Function" = Table.AddColumn(#"Removed Duplicates", "demo", each demo([ID])),
    #"Removed Columns1" = Table.RemoveColumns(#"Invoked Custom Function",{"ID"}),
    #"Expanded demo" = Table.ExpandTableColumn(#"Removed Columns1", "demo", {"ID", "A", "B", "C"}, {"ID", "A", "B", "C"})
in
    #"Expanded demo"
```

如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019114404788.png#pic_center)

##### 调用函数完成分列

选中参数表，然后按下图操作，调用我们此前建立的函数，并展开得到的列：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019114619393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

你也可以直接运行以下代码：

```
let
    Source = DATA,
    #"Removed Columns" = Table.RemoveColumns(Source,{"Number"}),
    #"Removed Duplicates" = Table.Distinct(#"Removed Columns"),
    #"Invoked Custom Function" = Table.AddColumn(#"Removed Duplicates", "demo", each demo([ID])),
    #"Removed Columns1" = Table.RemoveColumns(#"Invoked Custom Function",{"ID"}),
    #"Expanded demo" = Table.ExpandTableColumn(#"Removed Columns1", "demo", {"ID", "A", "B", "C"})
in
    #"Expanded demo"
```

结果如下，符合预期：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019114913863.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

### 测试

现在我们向数据源追加新的数据【D】：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019115239636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

运行查询，我们发现最新的列【D列】并未创建，这是因为在对NUMBER列进行展开时，其参数是固定的，我们需要让Table.ExpandTableColumn函数动态地获取完整的ID列表。这一步也不难，我们只需将参数表转换为列表，赋值给新变量【ID】：

```
ID = #"Removed Duplicates"[ID]
```

然后将原来代码的以下部分：

```
= Table.ExpandTableColumn(#"Removed Columns1", "demo", {"ID", "A", "B", "C"})
```

改成下方语句即可。

```
= Table.ExpandTableColumn(#"Removed Columns1", "demo", ID)
```

再次运行查询，所有列均按预期动态生成，问题得到完美解决。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019134243757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)


