---
layout: post
title:  PowerQuery 数据规范利器:Table.AddFuzzyClusterColumn
date:   2020-09-20 01:03:50 +0000
image:  12.jpg
tags:   [Power Query,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*本文讲解新M函数Table.AddFuzzyClusterColumn的强大功能以及用法。*

### 关于Table.AddFuzzyClusterColumn

Table.AddFuzzyClusterColumn是Power Query的表函数之一，它可以对数据进行模糊匹配并分组，从而规范数据源中的数据，什么意思呢？

一个简单的例子，比如地名“北京”，在数据源中它可能是“北 京”，“北京市”，“Beijing” 甚至“北平”，而该函数需要解决的，就是由数据录入不规范，数据本身的标准不统一等原因导致的这种数据杂乱的问题。为解决此问题，多数情况下，简单替换的方法显然不切实际，而该M函数可以使其规范化，这在数据清洗中特别实用。幸运的是，自Power BI Desktop上次更新（2020年8月）以来，已增加了对该函数的支持。下文讲解其具体用法。

*注：在CDS版本的PQ编辑器也可使用，但在Excel的PQ编辑器中暂不受支持（以Excel 2016为例）。*

### 基本用法

对于基本用法，我想引用[文档](https://docs.microsoft.com/en-us/powerquery-m/table-addfuzzyclustercolumn)中的举例的数据进行说明。首先我们有一个简单的表，里面包含英文城市温哥华和西雅图（如下），你可以留意到其中的单词大小写不一，且存在拼写错误（如seattl）:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916190349558.png#pic_center)

现在，我们回到Table.AddFuzzyClusterColumn函数，其用法如下：

```SQL
Table.AddFuzzyClusterColumn（
源表【必要】，目标列列名【必要】，规整后的列名【必要】，其他参数【可选】
）
```

因此，针对以上数据，编写M语句如下：

```SQL
= Table.AddFuzzyClusterColumn(
        #"Changed Type","Location","Location_Cleaned"
    )
```

【附】完整代码如下：

```SQL
let
  Source = Excel.Workbook(
      File.Contents("<路径>/示例文件0916.xlsx"), 
      null, 
      true
    ),
  Sheet1_Sheet = Source{[Item = "Sheet1", Kind = "Sheet"]}[Data],
  #"Promoted Headers" = Table.PromoteHeaders(Sheet1_Sheet, [PromoteAllScalars = true]),
  #"Changed Type" = Table.TransformColumnTypes(
      #"Promoted Headers", 
      { {"EmployeeID", Int64.Type}, {"Location", type text} }
    ),
  #"Result" = Table.AddFuzzyClusterColumn(#"Changed Type", "Location", "Location_Cleaned")
in
  #"Result"
```

这样，我们就可以得到如下结果，数据得到完美规范：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200916191933165.png#pic_center)


### 进阶用法

在进阶用法中，会在基本用法的基础上讲解参数的原理和使用方法，此外，处理的数据也改为文档中未涉及的中文文本。

首先利用以下M语句模拟一个火车站站名不规范的简易数据表：

```SQL
let
DATA = 
    Table.FromRecords(
        {
            [ID = 1, Location = "北京西站"],
            [ID = 2, Location = "北 京 西 站"],
            [ID = 3, Location = "广州东站"],
            [ID = 4, Location = "广 州 东 站"],
            [ID = 5, Location = "广州东火车站"],
            [ID = 6, Location = "西九龙站"],
            [ID = 7, Location = "西 九 龙 站"],
            [ID = 8, Location = "西九龙火车站"],
            [ID = 9, Location = "香港西九龙站"],
            [ID = 10, Location = "HK West Kowloon Railway Station"]
        },
        type table [ID = nullable number, Location = nullable text]
    )

in
    DATA
```

如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917175803852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)


按上文使用Table.AddFuzzyClusterColumn处理，此处使用第一个可选参数【Culture】，文档对该参数的定义是“允许根据区域性特定规则对记录进行分组”，其实就是该函数默认处理对象是英文字符，如果是其他语言，就应该使用Culture指明要处理的语言。于是：

```SQL
= Table.AddFuzzyClusterColumn(
        DATA,"Location","Location_Cleaned",[Culture="cn-ZH"]
    )
```

这样我们得到如下效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917175836800.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)



现在北京西站这个名称得到了规范，空格问题得到处理。但如何处理“广州东站zhan”这种混杂了拼音的记录呢？答案是使用强大的Threshold参数。Threshold参数的范围为0到1.0，默认值是0.8，数值为1代表原数据不做任何处理，其越低代表其在纠正数据时，对数据本身的容错率越高，此处将该参数设定为0.6后就解决了此问题。

*注：此处你还可以增加IgnoreSpace = true参数显式地忽略空格*

```SQL
= Table.AddFuzzyClusterColumn(
        DATA,"Location","Location_Cleaned",[Culture="cn-ZH",Threshold=0.6]
    )
```

效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917175951290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)



到此我们发现以上方法，还未能解决表中第六行至第十行的命名杂乱问题，PQ没有足够的依据去智能地给他们归类，因为PQ并不清楚西九龙是香港的一个地名，当然更不可能了解"HK West Kowloon Railway Station"的含义。为解决此问题，该函数引入了TransformationTable参数，这将允许我们自定义一个转换表，以使得PQ可以按照转换表的定义来规范数据，从而彻底解决此类问题。

首先，定义转换表：

```SQL
= Table.FromRecords(
        {
            [From = "西九龙站", To = "香港西九龙站"],
            [From = "西九龙火车站", To = "香港西九龙站"],
            [From = "HK West Kowloon Railway Station", To = "香港西九龙站"]
        },
        type table [From = nullable text, To = nullable text]
    )
```

然后再引用该参数：

```SQL
= Table.AddFuzzyClusterColumn(
        DATA,"Location","Location_Cleaned",[Culture="cn-ZH",Threshold=0.6,TransformationTable=TRANS_TABLE]
    )
```

这样我们就完美地解决了问题。效果如下：


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200917180039883.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

【附】完整代码为：


```SQL
let
DATA = 
    Table.FromRecords(
        {
            [ID = 1, Location = "北京西站"],
            [ID = 2, Location = "北 京 西 站"],
            [ID = 3, Location = "广州东站"],
            [ID = 4, Location = "广 州 东 站"],
            [ID = 5, Location = "广州东站zhan"],
            [ID = 6, Location = "香港西九龙站"],
            [ID = 7, Location = "香 港 西 九 龙 站"],
            [ID = 8, Location = "西九龙站"],
            [ID = 9, Location = "香港西九龙站"],
            [ID = 10, Location = "HK West Kowloon Railway Station"]
        },
        type table [ID = nullable number, Location = nullable text]
    ),    
TRANS_TABLE = 
    Table.FromRecords(
        {
            [From = "西九龙站", To = "香港西九龙站"],
            [From = "西九龙火车站", To = "香港西九龙站"],
            [From = "HK West Kowloon Railway Station", To = "香港西九龙站"]
        },
        type table [From = nullable text, To = nullable text]
    ),
DATA_CLEANED = 
    Table.AddFuzzyClusterColumn(
        DATA,"Location","Location_Cleaned",[Culture="cn-ZH",Threshold=0.6,TransformationTable=TRANS_TABLE]
    )
in
    DATA_CLEANED
```

### 总结

Table.AddFuzzyClusterColumn函数在数据源不规范时十分有用，掌握它的用法就可以轻松处理这类问题，但对于企业BI解决方案而言，通过ETL等方式从源头上解决数据杂乱的问题才是最规范的做法。
