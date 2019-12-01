---
layout: post
title:  用DAX强化图表交互力
date:   2019-03-23 09:03:50 +0300
image:  01.jpg
tags:   [Power BI,DAX]
---

## 简述
评价一个报表的好坏，至少有三个方面。一，是否符合业务需求；二，数据是否准确无误；三，用户使用的自由空间。这其中的第三点，许多人往往重视不足。本文介绍了我自己在提升报表交互力方面的工作中琢磨出的一点点小技巧，让PowerBI报表向智能化再进一步。

## 目标
1.可以让图表根据用户的选择，显示对应的字段的数据（跨列筛选）
2.让用户可以自由选择需要显示或需要移除的字段（以移动平均度量值为例）
3.允许用户通过传参改变图表数据背后的计算公式（以改变移动平均项数为例）

## 过程
数据与前文相同，比较简单，第一个目标就是用户可以选择下方红框的任意字段，来让图表显示对应字段的数据。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190323120443711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)
也许会有一个疑问,DAX在执行计算时，基于筛选上下文和行上下文，而没有“列上下文”，怎么跨列筛选呢？实际上，你会发现如果把这个所谓的列上下文用行上下文的形式表现出来就可以解决问题了，如下：

![*上表的“数量”后来改名叫“销售量”，懒得换图了*](https://img-blog.csdnimg.cn/20190323122512556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

通过设置变量“SELECTED"获取被筛选的值，然后利用SWITCH计算与该值相对应的度量值：

>```Python
>Measure = 
>VAR 
>SELECTED = IF(HASONEVALUE('模式'[模式]),VALUES('模式'[模式]),0)
>VAR
>AMT = sum(Data[销售额])
>VAR
>QTY = SUM(Data[数量])
>VAR
>RETURN
>SWITCH(SELECTED,"销售额",AMT,"销售量",QTY,"利润",PROFIT)
>```

效果如下（*看来这份数据在16年有几次严重的亏损*）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127163129207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

- [x] 可以让图表根据用户的选择，显示对应的字段的数据（跨列筛选）
- [ ] 让用户可以自由选择需要显示或需要移除的字段
- [ ] 允许用户通过传参改变图表数据背后的计算公式

下一步，为了案例需要，我创建了一个新的度量值”移动平均销售额“，以该度量值为例，让用户在图表决定显示或隐藏该数据。
*（该度量值以月为单位计算三项移动平均销售额）*

>```Python
>移动平均销售额 = 
>VAR
>X = 3
>RETURN
>CALCULATE(
>    AVERAGEX(
>        'Data',
>            CALCULATE(sum(Data[销售额]),
>                ALLEXCEPT('Data','Data'[订单日期]))),
>                    DATESINPERIOD('Data'[订单日期],LASTDATE('Data'[订单日期]),X,MONTH))
>```

实际上，实现这样的效果，还是利用了上述SWITCH的方法。多余不提，代码如下：

>```Python
>移动平均销售额 = 
>VAR
>SELECTED = IF(ISFILTERED('显示&隐藏'[显示&隐藏]),VALUES('显示&隐藏'[显示&隐藏]),"F")
>VAR
>X = 3
>VAR
>MOV_AVG = 
>CALCULATE(
>    AVERAGEX(
>        'Data',
>            CALCULATE(sum(Data[销售额]),
>                ALLEXCEPT('Data','Data'[订单日期]))),
>                    DATESINPERIOD('Data'[订单日期],LASTDATE('Data'[订单日期]),X,MONTH))
>RETURN
>SWITCH(SELECTED,"显示移动平均",MOV_AVG,"F",BLANK())
>```

效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127164000852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

- [x] 可以让图表根据用户的选择，显示对应的字段的数据（跨列筛选）
- [x] 让用户可以自由选择需要显示或需要移除的字段
- [ ] 允许用户通过传参改变图表数据背后的计算公式

最后，我们如果用户想看近四个月或近五个月的移动平均销售额呢？怎么才能让用户自由地设置这个度量值，接下来讲述PowerBI传参的方法（*事实上用户也可以自由选择以天为单位还是以周或月为单位，但设置原理与前述相同故不赘述*）
首先生成参数序列，有两种方法：
1. 在Power BI Desktop点击【Modeling】--【New Parameter】，在【What-if parameter】界面填写序列名，数据类型，最小与最大值，默认值和递增量，将自动生成序列表和序列度量值
2. 使用GENERATESERIES()函数新建序列表，然后使用SELECTEDVALUE()函数新建序列度量值

然后，将原代码的X=3改成刚刚设置的参数序列：

>```Python
>移动平均销售额 = 
>VAR
>SELECTED = IF(ISFILTERED('显示&隐藏'[显示&隐藏]),VALUES('显示&隐藏'[显示&隐藏]),"F")
>VAR
>X = '移动项数'[移动项数 Value]
>VAR
>MOV_AVG = 
>CALCULATE(
>    AVERAGEX(
>        'Data',
>            CALCULATE(sum(Data[销售额]),
>                ALLEXCEPT('Data','Data'[订单日期]))),
>                    DATESINPERIOD('Data'[订单日期],LASTDATE('Data'[订单日期]),X,MONTH))
>RETURN
>SWITCH(SELECTED,"显示移动平均",MOV_AVG,"F",BLANK())
>```

最后，用户就可以在下方红框处输入想要移动的项数，从而改变移动平均销售额的计算，效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127164420111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

- [x] 可以让图表根据用户的选择，显示对应的字段的数据（跨列筛选）
- [x] 让用户可以自由选择需要显示或需要移除的字段
- [x] 允许用户通过传参改变图表数据背后的计算公式

## 其他
此外，除了通过上述技巧，也可以使用第三方视觉对象来增强交互性。如果想自行开发自定义视觉对象，可以[点此](https://powerbi.microsoft.com/zh-tw/developers/custom-visualization/)到达Microsoft文档获取详情。





