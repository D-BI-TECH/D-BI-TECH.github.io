---
layout: post
title:  两种方法计算每月开发的新客户数量
date:   2019-03-24 20:03:50 +0000
image:  08.jpg
tags:   [Power BI,Power Pivot,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
---

## 简述
客户分析中，有时你可能需要分析每隔一段时间有多少新客户流入（*同样地，有多少老客户流失*），有时可能需要通过新客户开发数量来对员工绩效进行考核等等，对于此，本文分享了两种不同的DAX写法来计算新客户开发数。
## 过程
数据表(*部分*)如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190324214558811.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

<big>**方法一**

首先把订单按照客户ID和下单时间划分为"首单"和"非首单",然后计算每个月有多少个客户的订单被标记为"首单",那么这就代表每个月流入的新客户数量。
首先进行订单划分（*此处原理参考[本文](https://blog.csdn.net/qq_44794714/article/details/88776710)*）：

>```Python
>二次购买判断 = 
>VAR
>E_Date = 'Data'[订单日期]
>VAR
>CUST = 'Data'[客户 ID]
>RETURN
>IF(
>    SUMX(
>        FILTER('Data',CUST = 'Data'[客户 ID]&&E_Date > 'Data'[订单日期]),
>        COUNTROWS('Data'))>0,"非首次","首次")
>```

然后，计算新客户流入数量

>```Python
>每月新增客户数 = 
>CALCULATE(
>    DISTINCTCOUNT(Data[客户 ID]),'Data'[二次购买判断] = "首次")
>```

效果如下：

（*此图未指定年份，仅用于展示效果*）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127175115267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

<big>**方法二**

这种方法的原理类似于差分，用截至该月份为止的客户数减去到上个月为止的客户数得出该月的新增客户数，如需计算每季增加数，MONTH改为QUARTER即可：
（*关于[EXCEPT函数](https://docs.microsoft.com/en-us/dax/except-function-dax)*）

>```Python
>每月新增客户数 2 = 
>VAR
>X = -1
>VAR
>CUST = VALUES('Data'[客户 ID])
>VAR
>Pre_CUST = 
>CALCULATETABLE(VALUES('Data'[客户 ID]),
>    FILTER(ALL('Data'),
>        Data[订单日期]>DATEADD('Data'[订单日期],X,MONTH)
>        &&'Data'[订单日期]<MIN('Data'[订单日期])))
>RETURN
>COUNTROWS(
>    EXCEPT(CUST,Pre_CUST))
>```

效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127175556322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

## 其他

第一种方法不在公式内考虑时间间隔，而是在正常时间轴筛选下进行日，周，月或季的转换。第二种方法则需要在公式内部指定时间间隔，尽管正常的时间轴对它的筛选依然有效。当然，也可以通过[传参](https://blog.csdn.net/qq_44794714/article/details/88758445)的方式在可视化界面切换数据的时间间隔。