---
layout: post
title:  解决DAX中数据的循环迭代问题
date:   2019-07-04 06:03:50 +0000
image:  07.jpg
tags:   [Power BI,Power Pivot,SSAS,DAX]
---

一、循环迭代的难处
-----

实际业务中，数据迭代应用广泛，如在复利现值、折旧等方面的计算，然而众所周知的是，DAX是一个基于列引擎的函数语言，数据中每行基于其各自行上下文计算，但我们需要的效果是，列的第二行的数据基于第一行数值的计算产生，以此类推，列的第n行数据基于其第n-1行求得，这样的操作在Excel可以很轻松求出(如下图中的A、B、C三列)：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129192558511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

但在Power BI（或Power Pivot，SSAS Tabular）中，你虽然可以基于某个列利用诸如SUMX等函数进行迭代运算（如下所示的两种方法)：

>```Python
--法一
VAR _CAL = 
    CALCULATE(
        SUM('TABLE'[RENT]),
        FILTER('TABLE',
        'TABLE'[DATE]<=EARLIER('TABLE'[DATE])
        )
    )
--法二
VAR _SUMX = 
    SUMX(
        FILTER('TABLE',
        'TABLE'[DATE]<=EARLIER('TABLE'[DATE])),
        'TABLE'[RENT])
    RETURN COMBINEVALUE("-",_CAL,_SUMX)
>```

结果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129192615580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

但却难以基于某列的第一行的值，迭代该列自身而求出该列，换句话说，你可以很轻松依据A列的筛选上下文利用SUMX等类似的函数在B列求出其迭代结果（或在自定义的度量值得出结果）但却难以实现当某列只有第一行有数值时如何通过迭代其自身得出其第2至第n行的值，这样的计算在Excel很轻松，但用DAX却不易实现。

二、迭代的本质是数列
-----

事实上，DAX同样可以完美解决该问题，因为正如本段标题所言：迭代的本质是数列。当你用数列的思维去思考在Power BI的迭代问题，就能简单多了。因此，针对于上图A列的情况：当列的n+1行等于第n行加d时，整列的数据其实就是一个公差为2的等差数列，如下，我们只需要利用等差数列公式即可：

>```Python
_ITER_加法 = 
VAR D = 2
VAR A1 = 100
VAR N = 
CALCULATE(
    COUNTROWS('TABLE'),
    FILTER(
        'TABLE',
        'TABLE'[Date]<EARLIER('TABLE'[Date]))
)
RETURN
A1 + N * D  
>```

结果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129192625374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

同理，如果情况如同B列：n+1 = n * 2, 那么这就是一个等比数列：

>```Python
_ITER_加法 = 
VAR D = 2
VAR A1 = 100
VAR N = 
CALCULATE(
    COUNTROWS('TABLE'),
    FILTER(
        'TABLE',
        'TABLE'[Date]<EARLIER('TABLE'[Date]))
)
RETURN
A1 + N * D  
>```

结果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112919263540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

三、如何解决加减乘除类型的混合迭代
-----

正如图1的C列。当迭代公式类如“f(n +1) = (f(n) + d)*q”时，其逻辑就不像上面公式简单了，但实际上，这是一个数学问题了，本文在此不做拓展。其答案是：尽管此时数列f(n)本身是不规则数列，但f(n) – f(n-1)却是一个规则数列，而且是等比数列。依据这个思路，我们可以先利用等比数列规则求出f(n) – f(n-1)，然后反推出f(n)，实际上就是高中数学的差分法了。首先，f(n) – f(n-1)的计算公式为：

>```Python
F(n)-F(n-1) = 
VAR P_DATE = 'TABLE'[Date]
VAR D = 2
VAR Q = 2
VAR _RENT = 
    CALCULATE(
        FIRSTNONBLANK('TABLE'[RENT],0),
        'TABLE')
VAR _FIXED_DISTANCE = (_RENT+D)*Q - _RENT
VAR _LOOP_1 = GENERATESERIES(1,COUNTROWS('TABLE'))
VAR _LOOP_2 = 
    ADDCOLUMNS(_LOOP_1,"_FIXED_DISTANCE",
        _FIXED_DISTANCE*PRODUCTX(
            FILTER('TABLE','TABLE'[Date]<P_DATE),Q))
VAR _MAX = MAXX(_LOOP_2,_RENT)
VAR VARIABLE_DISTANCE = 
    IF(
        ISBLANK(
            MAXX(
                FILTER(_LOOP_2,_RENT=_MAX),
                [_FIXED_DISTANCE])),
        _FIXED_DISTANCE,
        MAXX(
            FILTER(_LOOP_2,_RENT=_MAX),
            [_FIXED_DISTANCE]))
RETURN
VARIABLE_DISTANCE
>```

注1：以如图1的C列为例，其公差为3，公比为2，则f(n) – f(n-1)为首项为f(2)-f(1),公比为2的等比数列
注2：第14、15行处，由于差分法会使第一项留空，故使用IF(ISBLANK(),,)使第一行有值

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129192645754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

在得出f(n) – f(n-1)后，即可反推出f(n)的值，因为f(n)是一个以f(n) – f(n-1)为公差的不等差数列，计算如下：

>```Python
_ITER_加乘法混合 = 
VAR P_DATE = 'TABLE'[DATE]
VAR _RENT = 
    CALCULATE(
        FIRSTNOBLANK('TABLE'[RENT],0),
        ALL('TABLE')
    )
VAR _LOOP_1 = GENERATESERIES(1,COUNTROWS('TABLE'))
VAR _LOOP_2 = 
    ADDCOLUMNS(_LOOP_1,"RENT",
    _RENT+SUMX(FILTER('TABLE',
    'TABLE'[DATE]<P_DATE),
    'TABLE'[F(n)-F(n-1)])
    )
VAR _MAX = MAXX(_LOOP_2,_RENT)
VAR _RESULT = 
    MAXX(FILTER(_LOOP_2,_RENT=_MAX),
    [_RENT])
RETURN _RESULT
>```

结果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129192653470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这样我们就成功完成了在这种复杂公式之下的循环迭代。经测试，无论我们在公式中如何改变D和Q的值，该公式都返回了正确的循环迭代结果。

*注：本文首发于PowerBI星球公众号，案例源文件已由采悟老师分享至PowerBI星球-知识星球*