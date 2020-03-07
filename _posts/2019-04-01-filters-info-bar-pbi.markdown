---
layout: post
title:  用DAX构建筛选信息栏
date:   2019-04-01 06:03:50 +0000
image:  08.jpg
tags:   [Power BI,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

## 简述
本文主要介绍如何使用DAX构建筛选信息栏。筛选信息栏，就是能够在整个报表或某个页面以可视文本形式展示目前数据所被应用的筛选上下文，其作用主要有二：

1. 展示当前页面或特定可视化控件的筛选上下文，以便于进行数据及DAX公式的验证与错误排查；
2. 便于用户掌握目前数据的筛选情况，当筛选器较多，经常需要用到多选或不同的可视化控件应用不同的筛选条件时十分有用。

本案例目标是在对下表进行筛选时，让信息栏显示当前"城市"、"子类别"及"月份"的筛选信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331160255866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

## 过程
实现这个功能实际上不需要自己写代码，因为如果你有使用过[DAX Studio](https://daxstudio.org/),你会发现这个功能已经被内置其中，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331224701278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

代码会在右方自动生成，可直接根据你的个人需要进行修改，快捷省时：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331224812335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

对于本案例我们只需要"城市"、"子类别"及"月份"的筛选信息，如下：

>```Python
>Filter_Test = 
>VAR Filters_Limit = 5
>RETURN
>    IF ( 
>    ISFILTERED ( 'Data'[城市] ), 
>    VAR ___f = FILTERS ( 'Data'[城市] ) 
>    VAR ___r = COUNTROWS ( ___f ) 
>    VAR ___t = TOPN ( Filters_Limit, ___f, 'Data'[城市] )
>    VAR ___d = CONCATENATEX ( ___t, 'Data'[城市], ", " )
>    VAR ___x = "[城市]: " & ___d & IF(___r > Filters_Limit, ", ... [" & ___r & " 项已被选定]") & " " 
>    RETURN ___x & UNICHAR(13) & UNICHAR(10)
>    )&
>    IF ( 
>    ISFILTERED ( 'Data'[子类别] ), 
>    VAR ___f = FILTERS ( 'Data'[子类别] ) 
>    VAR ___r = COUNTROWS ( ___f ) 
>    VAR ___t = TOPN ( Filters_Limit, ___f, 'Data'[子类别] )
>    VAR ___d = CONCATENATEX ( ___t, 'Data'[子类别], ", " )
>    VAR ___x = "[子类别]: " & ___d & IF(___r > Filters_Limit, ", ... [" & ___r & " 项已被选定]") & " " 
>    RETURN ___x & UNICHAR(13) & UNICHAR(10)
>    )&
>    IF ( 
>    ISFILTERED ( 'Date'[月份] ), 
>    VAR ___f = FILTERS ( 'Date'[月份] ) 
>    VAR ___r = COUNTROWS ( ___f ) 
>    VAR ___t = TOPN ( Filters_Limit, ___f, 'Date'[月份]  )
>    VAR ___d = CONCATENATEX ( ___t, 'Date'[月份] , ", " )
>    VAR ___x = "[月份]: " & ___d & IF(___r > Filters_Limit, ", ... [" & ___r & " 项已被选定]") & " " 
>    RETURN ___x & UNICHAR(13) & UNICHAR(10)
>    )
>```

效果如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112814334849.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这里面，需要指出的是，UNICHAR(13) & UNICHAR(10)在这里相当于换行符（换行+回车），因此你可以像如下这样，设置换行的条件，如下修改后的公式(以“城市”为例，其他同)，只有当该行数据字符长度超过20时才会换行：
（*有关于[UNICHAR](https://docs.microsoft.com/zh-hk/dax/unichar-function-dax),可以点击[这里](https://en.wikipedia.org/wiki/List_of_Unicode_characters)查看Unicode字符列表，还有很多待挖掘的有用字符*）

>```Python
>VAR Width_Limit = 20
>VAR Filters_Limit = 5
>RETURN
>    IF ( 
>    ISFILTERED ( 'Data'[城市] ), 
>    VAR ___f = FILTERS ( 'Data'[城市] ) 
>    VAR ___r = COUNTROWS ( ___f ) 
>    VAR ___t = TOPN ( Filters_Limit, ___f, 'Data'[城市] )
>    VAR ___d = CONCATENATEX ( ___t, 'Data'[城市], ", " )
>    VAR ___x = "[城市]: " & ___d & IF(___r > Filters_Limit, ", ... [" & ___r & " 项已被选定]") & " " 
>    RETURN if(len(___x)>Width_Limit, ___x & UNICHAR(13) & UNICHAR(10),___x)
>```

这样，只有当某个筛选信息长度超过20个字符时，才会进行换行操作。根据报表实际情况设置宽度限制以使得信息栏能够充分利用空间。如下，城市字段的筛选信息超过了限制长度，因此其后被加上换行符：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128144449725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

最后，还需要改进的是，对于月份这种时间型筛选，我们需要一种更人性化的阅读体验，比如，选择1月、2月、3月、4月，对应显示1 - 4月(或Jan - Apr):

>```Python
>VAR MONTH_TABLE =
>    SUMMARIZE ( 'DATE', 'Date'[月份_num], 'DATE'[月份] )
>VAR DATE_RESULT = 
>    CONCATENATEX(MONTH_TABLE,
>        VAR _MON = 'Date'[月份]
>        VAR _MONNUM = 'Date'[月份_num]
>        VAR ISNEXTSELECTED =  _MONNUM + 1 IN MONTHNUM
>        VAR ISPRESELECTED =  _MONNUM - 1 IN MONTHNUM
>        RETURN
>            IF(
>                NOT(ISPRESELECTED && ISNEXTSELECTED),
>                _MON & IF(ISNEXTSELECTED,"-",",")
>            ),
>        "",
>        'Date'[月份_num]
>    )
>    RETURN
>    LEFT(DATE_RESULT,LEN(DATE_RESULT)-1)
>```

利用这个公式，就可以让DAX判断不同的所选月份是否连续，进而组合成我们想要的效果。这里公式也许看起来复杂，但清楚这里的逻辑后就很容易懂了，如下表所示(以选择6月为例)：


| 筛选情况： | 仅前一个月份被选 |  仅后一个月份被选| 前后月份都被选择 | 前后都无月份被选 |
|--|--|--|--|--|
| 返回结果 | 6 ， | 6 - | （blank） | 6 |
| 返回结果(英文简写) | Jun ， | Jun - | （blank） | Jun |

*(注1：表中的“前后”指连续的前后)*
*(注2：当5,6,7三个月份被选择时，由于对于6月来讲前后月份都被选择，因此不显示6月，因此结果显示"5 - 7"*）

因此完整代码如下：

>```Python
>选定项信息栏 = 
>VAR Width_Limit = 20
>VAR Filters_Limit = 5
>VAR MONTHNUM = VALUES('Date'[月份_num])
>VAR MONTH_TABLE =
>    SUMMARIZE ( 'DATE', 'Date'[月份_num], 'DATE'[月份] )
>VAR DATE_RESULT = 
>    CONCATENATEX(MONTH_TABLE,
>        VAR _MON = 'Date'[月份]
>        VAR _MONNUM = 'Date'[月份_num]
>        VAR ISNEXTSELECTED =  _MONNUM + 1 IN MONTHNUM
>        VAR ISPRESELECTED =  _MONNUM - 1 IN MONTHNUM
>        RETURN
>            IF(
>                NOT(ISPRESELECTED && ISNEXTSELECTED),
>                _MON & IF(ISNEXTSELECTED,"-",",")
>            ),
>        "",
>        'Date'[月份_num]
>    )
>RETURN
>    IF ( 
>    ISFILTERED ( 'Data'[城市] ), 
>    VAR ___f = FILTERS ( 'Data'[城市] ) 
>    VAR ___r = COUNTROWS ( ___f ) 
>    VAR ___t = TOPN ( Filters_Limit, ___f, 'Data'[城市] )
>    VAR ___d = CONCATENATEX ( ___t, 'Data'[城市], ", " )
>    VAR ___x = "[城市]: " & ___d & IF(___r > Filters_Limit, ", ... [" & ___r & " 项已被选定]") & " " 
>    RETURN if(len(___x)>Width_Limit, ___x & UNICHAR(13) & UNICHAR(10),___x)
>    )&
>    IF ( 
>    ISFILTERED ( 'Data'[子类别] ), 
>    VAR ___f = FILTERS ( 'Data'[子类别] ) 
>    VAR ___r = COUNTROWS ( ___f ) 
>    VAR ___t = TOPN ( Filters_Limit, ___f, 'Data'[子类别] )
>    VAR ___d = CONCATENATEX ( ___t, 'Data'[子类别], ", " )
>    VAR ___x = "[子类别]: " & ___d & IF(___r > Filters_Limit, ", ... [" & ___r & " 项已被选定]") & " " 
>    RETURN if(len(___x)>Width_Limit, ___x & UNICHAR(13) & UNICHAR(10),___x)
>    )&
>    "[月份]: " & LEFT(DATE_RESULT,LEN(DATE_RESULT)-1)
>```

最后效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128143118268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

## 其他
本案例进一步展示了DAX语法的灵活性。关于本文，需要感谢BI大神Marco在此前发的[Blog](https://www.sqlbi.com/articles/displaying-filter-context-in-power-bi-tooltips/)，他在这篇文章里介绍了将筛选上下文显示于特定控件的Tooltips的方法，这也是本案例的思路来源。

