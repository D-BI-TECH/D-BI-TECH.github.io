---
layout: post
title:  PowerBI：关于PBIX，PBIT及PBIDS
date:   2020-10-31 01:03:50 +0000
image:  18.jpg
tags:   [Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*我们知道PowerBI不仅有PBIX文件类型，还有PBIT和PBIDS，这些文件的作用是什么，互相间的区别是什么，国内尚无任何相关资料，下文将就此做详细介绍*

（封面：船底座星云）

### Power BI文件简介

Power BI目前主要有三种文件类型，PBIX，PBIT以及PBIDS。

- PBIX 这是最常用的Power BI报表文件，.pbix延用了O365家族对文件后缀的命名习惯（如Word文档.docx,Excel文件.xlsx等等）
- PBIT 全称Power BI Template文件，是早在2016年就已推出的Power BI文件格式
- PBIDS 全称Power BI DataSource文件，于2019年10月推出（其中RS版PBI自今年十月起支持该文件类型）

具体可以参考我制作的表格：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031173754352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)


从上表可知，其中PBIX文件保存了完整的Power BI报表内容，PBIT则是除了不包含数据本身外，其他方面都有包含，而PBIDS则仅保留了数据源的连接凭据。

*在Power BI Service以及PBIRS，还存在一个RDL文件，但该文件属于分页报表文件，不在本文的技术栈范畴*

### PBIT以及它的用途

首先我们讲讲PBIT文件，意为Power BI临时文件，它保留了Power BI前后端完整的设置，但不包含数据本身，任何PBIX报表都可以另存为PBIT文件。它有两种打开方式，一是直接双击运行PBIT文件，二是首先打开PBID，然后导入PBIT文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031165840589.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

由于PBIT不仅包含数据源凭据，还保留了原PBIX在PQ层的数据编辑，因此导入PBIT文件后会直接开始导入数据，如果在此前的PBIX的PQ编辑器中设置了查询参数，此处在导入数据前会有提示框让用户输入参数，然后再依据选择的参数导入对应的数据。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031171243910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

因此当我们需要在组织内按区域经理或者按照子公司等其他参数字段向下分发统一格式的PBIX报表时，利用PBIT将十分有用，特别是由于其不包含数据本身，报表之间的共享变得十分容易，因为你可以轻松地把PBIT文件附加在邮件或共享盘中，而不是花较多时间把一个高达10GB的报表文件上传到云盘。

PBIT文件给我们带来的另一个好处是，报表开发的团队协作变得高效，你可以把你未开发完成的报表（或者说你只需要完成后端的数据建模部分）交给其他同事继续开发和改进，且修改保存后的PBIX文件自然会独立于原来的PBIX文件，由于我们已知道，PBIT文件不占用磁盘空间，因此，报表可以这样不断地进行版本迭代，提高协作效率的同时，还有利于报表的版本管理。

*使用DirectQuery的报表文件也可以另存为PBIT，但对于完全使用DirectQuery模式的报表并没有必要这样做*

### PBIDS以及它的用途

相比于PBIT，PBIDS文件则简陋了许多，它仅仅包含数据源凭据。同样的，任何PBIX文件都可以另存为PBIDS文件格式供后续使用，但它不在【另存】处导出，而是在数据源设置里操作。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031174604905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

这里不得不提一下PBIDS文件的一个尴尬的限制，就是它只允许保存一种数据源凭据，如图，当我准备同时导出Excel数据源和SQL Server数据源凭据时，导出为PBIDS的选项变得不可用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031175229448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

这的确限制了该文件格式的用武之地，有时对于一些对数据源不了解的开发者，如果从拥有全部数据源凭据的报表开始做起将会很便利，如果只能保存一种数据源，倒不如直接使用PBIT文件更好些。此外，这里的数据源凭据也仅仅是数据源定义，对于需要输入密码的数据源，打开PBIDS后还是要输密码的。

虽然存在这些问题，但PBIDS实际只是一个结构极其简单的JSON文件，可以使用文本编辑器进行编辑，甚至可以使用程序自动生成对应的PBIDS文件。


```SQL
{
  "version": "0.1",
  "connections": [
    {
      "details": {
        "protocol": "tds",
        "address": {
          "server": "localhost\\sqlexpress",
          "database": "ContosoRetailDW"
        },
        "authentication": null,
        "query": null
      },
      "options": {},
      "mode": null
    }
  ]
}
```

当然，如果你要自己直接编写PBIDS，或者使用脚本生成，需要注意内容格式的小细节，比如路径处的反斜杠必须双写，否则PBID打开该文件时很可能报错。

### 拓展：PBIX还可以怎么用

在讲解完PBIT和PBIDS后，回归PBIX。PBIX除了在开启增强元数据集后可以和PBIT文件一样通过解压缩导出模型语义DataModelSchema之外（这里很多博主都讲过），还可以把PBIX分为数据集和报表两层，这使得在PBI报表开发中，前端可以从后端单独分离出来，便于组织协作。关于此，微软其中一位MVP--Reid Havens介绍了一种简单粗暴的方法，参考[此视频](https://www.youtube.com/watch?v=PlrtBm9YN_Q)。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031182419765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)





