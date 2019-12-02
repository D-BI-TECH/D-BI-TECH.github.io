---
layout: post
title:  用DAX实现20/80(帕累托)分析
date:   2019-03-19 14:03:50 +0000
image:  02.jpg
tags:   [Power BI,Power Pivot,DAX]
---

## 简述&目标
**本文将使用DAX解决以下计算：**
1. 实现对产品的AB分类，找出总共贡献利润占总利润80%左右的那些产品
2. 计算出排名前20%的订单所贡献的销量(销售额或利润）占总销量(销售额或利润）的百分比
3. 计算出排名前20%的客户所贡献的销量(销售额或利润）占总销量(销售额或利润）的百分比

## 方法
对于以上所有计算，我们分别使用不同的方法。其中第一个我们使用Marco Russo的方法，（对所有产品进行ABC分类，和本文的AB分类同理）可以点击[这里](https://www.daxpatterns.com/abc-classification/)看原版操作。对于第二和第三个计算目标，我会在本文分享我自己的方法。

## 过程
数据还是原来的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329171639445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

<big>**1.实现对产品的AB分类，找出总共贡献利润占总利润80%左右的那些产品**

首先，新建一个计算列，算出累计利润：

>```Python
>利润_Column = 
>sumx(filter('Data','Data'[利润]>=EARLIER('Data'[利润])),'Data'[利润])
>```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329174140468.png)

以上的计算逻辑是，对于利润的第一行等于3783.78,filter会在表中迭代寻找大于等于这个值的行，结果只找到了它本身，因此在[利润_Column]中的计算结果就是3783.78，以下计算以此类推，从而使该计算列得出累积利润（*注：我们没有在公式中进行任何排序操作，但DAX的运算逻辑却能够在计算累积的时候严格按照从大到小的顺序把数据累加了*）

接下来再进行分类，累积利润小于80%的，即利润排名靠前一共创造接近或等于80%利润的那些产品，会被划分为A类产品，其余产品划为B类：

>```Python
>20/80 AB分类 = 
>var
>accum_precent = 
>divide('Data'[利润_Column],sum('Data'[利润]))
>return
>if(
>accum_precent <= 0.8,"A","B")
>```

效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190329175538291.png)

<big>**2.计算出排名前20%的订单所贡献的销量(销售额或利润）占总销量(销售额或利润）的百分比**

先定义我们要计算的字段：销量，销售额或利润（此处详见[本文]({{site.baseurl}}/enhance-Chart-Interactivity/)关于根据用户选择决定度量值计算字段的部分）

>```Python
>Measure = 
>VAR 
>SELECTED = IF(HASONEVALUE('模式'[模式]),VALUES('模式'[模式]),0)
>VAR
>AMT = sum(Data[销售额])
>VAR
>QTY = SUM(Data[数量])
>VAR
>PROFIT = SUM(Data[利润])
>RETURN
>SWITCH(SELECTED,"销售额",AMT,"销售量",QTY,"利润",PROFIT)
>```

然后，下方代码会计算出排名前20%的所有订单的销售额（或销售量，利润,下同)，然后用它除以总销售额，从而算出金额排名前20%的所有订单占总销售额的比例：

>```Python
>20/80 order %= 
>var
>percent_ = 0.2
>var
>f_table = ALLSELECTED('Data')
>var
>all_amt = CALCULATE('Data'[Measure],f_table)
>var
>t20p = 
>CALCULATE('Data'[Measure],
>    topn(
>        ROUND(
>            COUNT('Data'[订单 ID])*percent_,0),
>            f_table,'Data'[Measure],DESC)
>)
>return
>divide(t20p,all_amt,BLANK())
>```

这里的计算则和第一种明显不同，我们为主表指定了排名，[TOPN()](https://docs.microsoft.com/en-us/dax/topn-function-dax)会根据[Measure]降序排名然后取出排名前20%的部分，然后这部分数据传递给CALCULATE成为其计算'Data'[Measure]的筛选上下文，因此变量t20p即得到排名前20%的所有订单的销售额（或销售量，利润），让它除以总销售额，即得出这部分订单所贡献的销售额（或销售量，利润）占比，效果见文末截图。  
*注：由于存在部分负利润订单，利润排名前20%的订单占总利润可能超过100%*  

<big>**3.计算出排名前20%的客户所贡献的销量(销售额或利润）占总销量(销售额或利润）的百分比**

这个问题的计算原理和上一个相同，但在这里的计算方法和上一个稍有不同，因为这里**调用了虚拟表的列**：

>```Python
>20/80 Customers %= 
>var
>percent_ = 0.2
>var
>all_amt = CALCULATE('Data'[Measure])
>var
>summ = SUMMARIZE('data','Data'[客户 ID],"cccc",'Data'[Measure])
>var
>top_table = 
>topn(
>    round(
>         DISTINCTCOUNT('Data'[客户 ID])*percent_,0),summ,[cccc],DESC)
>return
>divide(
>CALCULATE(sumx(top_table,[cccc])),
>all_amt,
>BLANK())
>```

上方我创建了一个名为summ的虚拟表，它以客户为维度，包括其中的一个虚拟列“cccc",然后使用topn得出贡献销售额（销量或利润）排名前20%的所有客户，赋值给变量”top_table",然后使用CALCULATE(sumx(top_table,[cccc]))得出排名前20%的客户所贡献的销售额（销量或利润）,再除以总销售额（销量或利润）最终得出结果。（*注：这里之所以使用虚拟表summ而非建立一个真实存在的客户表，是因为它完全动态，并且有更好的代码运行效率，同时我们也避免了额外建立表格关系这样的麻烦*）
效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112814012443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128140133139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128140139943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

到这里，可能有人会说，我要的其实不仅是这个占比，重要的是要把这些排名前20%的客户找出来。这个问题其实反而不用那么麻烦，尽管对上面的代码做一点修改也可以做到，但代码没必要那么冗长，这里只需直接使用TOPN()筛选一下再算行数即可：

>```Python
>TOP 20% CUST = 
>CALCULATE(
>    COUNTROWS(Data),
>        topn(round(
>            DISTINCTCOUNT(Data[客户 ID])*0.2,0),
>                Data,'Data'[Measure],DESC))
>```

把这个度量值拉倒筛选栏即可找出TOP 20%的客户

## 其他
本文中，一个很重要的点是调用虚拟表的列，这个技术如果能灵活掌握，相信DAX的使用水平可以进步很多。此外，对于二八法则分析，实际业务中当然不会仅限于客户、订单和产品方面，但只要能活用这些方法，即可应形于无穷。