---
layout: post
title:  DAX虚拟连接函数TREATAS()用法介绍
date:   2019-03-27 06:03:50 +0000
image:  09.jpg
tags:   [Power BI,Power Pivot,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

## 简述
[TREATAS()](https://docs.microsoft.com/en-us/dax/treatas-function)函数是DAX中用于表格间虚拟连接的函数,当遇到如下情况时，可以考虑使用TREATAS：
1.维度表或事实表之间没有可以单独关联的列；
2.出现多对多或其他无法使用直接的方法关联的情况
3.数据模型非常复杂时，通过建立虚拟关系以减少对表格之间物理连接的依赖
 *（根据Marco的说法：额外的物理关系可能会在过滤器传播到其他表时产生某种副作用）*

## 过程
本案例数据是如下两张没有关联结构完全相同的销售表：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128103135224.png)

左边的是主表，右边的叫副表，很显然它们都没有唯一列，无法直接建立关联。假设我们希望使用副表的城市字段来筛选主表的利润，我们可以使用如下方法在主表新建度量值：

>```Python
>利润_Treatas = 
>CALCULATE(sum('Data'[利润]),
>    TREATAS(VALUES(copy[城市]),'Data'[城市]))
>```

结果如下(*注意：下图城市这一字段来自和主表没有任何关联的副表*)：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128101346707.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

但当我们增加副表的[客户ID]做筛选时，对应的数据却没有被正确计算，这是因为[客户ID]并没有利用Treatas和主表建立关联（*但可以使用[地区]这一字段，因为它是[城市]的上层字段*)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128101907432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

解决方法如下：

>```Python
>利润_Treatas = 
>CALCULATE(sum('Data'[利润]),
>    TREATAS(SUMMARIZE('Copy','Copy'[城市],'Copy'[客户 ID]),
>    'Data'[城市],Data[客户 ID]))
>```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128102025758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

## 其他
Treatas()之所以可以实现虚拟连接，是因为它可以把本表指定列的筛选上下文属性传递至目标表,因而目标表可以获得对应的筛选能力（*如本例，城市和客户ID字段的筛选上下文被传递至SUMMARIZE后的副表，因此副表的对应列可以筛选本表的指定值*）