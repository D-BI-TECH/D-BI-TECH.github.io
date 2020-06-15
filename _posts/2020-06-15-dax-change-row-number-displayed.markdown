---
layout: post
title:  DAX应用：控制表格显示行数
date:   2020-06-15 03:03:50 +0000
image:  10.jpg
tags:   [DAX,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

（本文要解决的问题：利用DAX度量值，控制前端表格显示行数）

### 需求描述

如下图，是一张设备采购与折旧记录表，需求是当用户选择任一单号，或多选以及全选，可以提供一个勾选项，使表格仅显示一行数据，至于该行是日期最近的一行还是金额最大的一行，提供一个单选框由用户决定。如下，选择了NO_121，采购项是戴尔笔记本电脑，用户可以选择依据采购日期，仅显示离当日最近的一行数据（即下图最后一行）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200615134002518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 解决思路

将问题分解。首先实现无论选择任何单号，多选或全选，通过切片器控制使表格仅显示离当日最近的一行数据。最后，提供一个切片器，让用户控制该行是日期最近的一行还是金额最大的一行。

对于第一个问题，有两种方案，其一是对原表建立一个索引列，该列按采购日期逐行递增，可以在SQL层完成，也可以直接利用PowerQuery的[Table.AddIndexColumn](https://docs.microsoft.com/en-us/powerquery-m/table-addindexcolumn)函数增加索引列，于是可以利用DAX建立如下度量值：

```SQL
MAX_INDEX = 
VAR MAX_1 = CALCULATE(MAX('Dataset'[Index]),
	FILTER(ALLEXCEPT('Dataset','Dataset'[No]),
	NOT ISBLANK(SUM('Dataset'[Amount（发生额）]))))
RETURN 
MAX_1
```

建立参数表用作前端单选框：

```SQL
PARA_SHOWLASTENTRY = DATATABLE("NAME",STRING,{{"Show last only"}})
```

然后利用Calculate设置度量值使之仅显示索引最大的那行,即在当前上下文环境下日期最大的那行：

```SQL
AMT_ONE = 
IF(ISFILTERED(PARA_SHOWLASTENTRY[NAME]),
    IF(MAX('Dataset'[index]) = [MAX_INDEX],[AMT],BLANK())
    ,[AMT])
```

这样在效果上可以满足需求：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200615142847484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

但它不是最好的方案。特别是当我需要解决第二个问题时，需要依据金额而非日期来显示时，就需要再新建一个索引列，且此方法不能满足用户仅显示N行的定制需求。下文会讲到解决此需求的更合适的思路。

### 最终方案

仅显示一行或N行，并且可以依据不同的字段来决定显示的规则，使用RANKX函数再合适不过。

首先设定一个参数表：

```SQL
PARA_BY = 
DATATABLE("NAME",STRING,"VALUE",INTEGER,
{
    {"依据 Posting Date",0},
    {"依据 AMT(金额)",1}
})
```

将PARA_BY的NAME字段作为单选切片器来控制VALUE字段的值。利用该值使用SWITCH函数来实现依据过账日和金额的排名值切换：

```SQL
_RANK =
SWITCH (
    SELECTEDVALUE ( PARA_BY[VALUE] ),
    0, RANKX (
        ALLSELECTED ( 'Dataset' ),
        FIRSTNONBLANK ( 'Dataset'[Posting Date（过账日）], 0 ),
        ,
        DESC,
        SKIP
    ),
    1, RANKX ( ALLSELECTED ( 'Dataset' ), [AMT],, DESC, SKIP )
)

```

最后，引用该度量值，排名等于1的就显示，否则为空：

```SQL
_BY_RANK = IF([_RANK]=1,[AMT],BLANK())
```

将_BY_RANK作为金额值拖入表格控件，由切片器控制，效果如下：

- 默认状态：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200615154253691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

- 选择"依据AMT(金额)":

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061515434129.png)

- 选择"依据Posting Date":

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020061515443219.png)
更进一步地，也可以将_BY_RANK度量值中的1设为变量(what-if参数)，交由用户设定：

```SQL
_BY_RANK = IF([_RANK]<='ROWS'[ROWS Value],[AMT],BLANK())
```

这样就能够在前端实现控制表格显示的行数以及显示依据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200615155242230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)