---
layout: post
title:  在多对一关系中正确计算数据
date:   2019-03-19 10:03:50 +0300
image:  02.jpg
tags:   [Power BI,Power Pivot,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
---


多对一关系是PowerBI表格模型中最常见的关系之一，通常，我们利用related()从唯一端把目标数据拖到多端，但有时却会产生计算错误，本文结合实际应用场景，通过一个在销售表中正确显示库存数量的例子，来一步步地解决这个错误。

## 案例概览
在实际业务中，销量数据和库存数据放在同个表已经很常见，本案例的需求是使销售和库存在powerbi的同一个表格控件中展示，库存可以根据需求切换成“店铺库存”模式或“仓库库存”模式，在实际的应用场景中，表格的关系一旦复杂，对库存数据的计算就可能会出错，本文大幅简化了表格模型，找出其中的问题并给出解决方案。
本例数据表格关系如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019031911075910.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)


## 数据介绍  
Sales作为销售数据表，WH_stock是仓库库存，SR_stock是各个店铺库存，Shop则是店铺信息。字段方面，[No]是单号，[ItemNo.]是产品号，[ShopName]是店铺名称。
sales表和每个表格的关联字段都不一样：WH_stock和Sales通过[ItemNo]+[size]进行一对多关联，而SR_stock和sales表是通过[ItemNo]+[size]+[ShopName]进行一对多关联，Shop和Sales是通过[ShopName]一对多关联。

## 解决方法
首先，我做了一个MODE表，当用户选择SR时，Stock显示店铺库存，当用户选择WH时，其显示仓库库存，选择ALL则是总库存，表达式如下：

>```Python
>Stock = 
>VAR
>WH = AVERAGE('WH_stock'[StockQty])
>VAR
>SR = AVERAGE('SR_stock'[StockQty])
>VAR
>AL = WH + SR
>VAR
>TF = IF(HASONEVALUE('MODE'[MODE]),VALUES('MODE'[MODE]),0)
>RETURN
>SWITCH(TRUE(),TF="WH",WH,TF="SR",SR,TF="ALL",AL)`
>```

结果出现如下Stock数据并没有显示产品本身的库存

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127154042533.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这个错误很好解决，这是因为两个库存表的库存数据[StockQty]的筛选上下文没有被传递至sales表中，我们只需要使用related就可以解决：

>```Python
>Stock = 
>VAR
>WH = AVERAGE('Sales'[WH_StockQty]) #related from 'WH_stock'  
>VAR
>SR = AVERAGE('Sales'[SR_StockQty]) #related from 'WH_stock'  
>VAR
>AL = WH + SR
>VAR
>TF = IF(HASONEVALUE('MODE'[MODE]),VALUES('MODE'[MODE]),0)
>RETURN
>SWITCH(TRUE(),TF="WH",WH,TF="SR",SR,TF="ALL",AL)
>```

结果如下：  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127154217347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

可能你很快发现能发现第二个错误是库存数据被日期和城市两个字段筛选了，事实上我们只希望它用于筛选销量数据[SalesQty]而不希望库存也受影响，这种情况下，我们移除在stock度量值中[Date]和[City]两个字段的筛选上下文就可以解决问题：

>```Python
>Stock = 
>VAR
>WH = CALCULATE(AVERAGE('Sales'[WH_StockQty]),all(Sales[Date]),all(Shop[City]))
>VAR
>SR = CALCULATE(AVERAGE('Sales'[SR_StockQty]),all(Sales[Date]),all(Shop[City]))
>VAR
>AL = WH + SR
>VAR
>TF = IF(HASONEVALUE('MODE'[MODE]),VALUES('MODE'[MODE]),0)
>RETURN
>SWITCH(TRUE(),TF="WH",WH,TF="SR",SR,TF="ALL",AL)
>```

如下，在保证销售数据能够正常筛选的同时，库存数据不受影响

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127154953452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

然而，排错到此还未结束，如果你打开总计，你会发现stock的总计不可能只有82，显然，它也受到了同样的行上下文影响
如果average改成sum，则可以解决这个问题，但它也引发了各个产品的库存数量被重复计算，这最终也会导致总计被夸大：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127155124933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

比如A127（12CM）这个产品的实际总库存为60，却被错误记为279，这是由于同一个产品在sales表中有多个订单记录，因此如下表所示，库存被错误计为：53 * 5+7 * 2，如何解决这个问题？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190319131224944.png)

有一个办法可以解决这个问题，可以称其为“对折法”，库存数据在sales表的上下文环境下被重计，我们可以计算每个产品被重计的次数，然后除以它，把库存数据折回成原始值：
首先，分别在店铺库存表（SR_stock）和仓库库存表（WH_stock）建立物理计算列：

>```Python
>EXPAND_SR = countrows(RELATEDTABLE('Sales'))
>EXPAND_WH = countrows(RELATEDTABLE('Sales'))
>```

以上两个计算列中，relatedtable利用了sales表的筛选上下文，通过逐行的迭代计算，就可以count出库存表里每个Item在sales表中被重计的次数,然后，回到sales表，用里面的库存列除以这个次数：

>```Python
>Stock =
>VAR WH =
>    CALCULATE (
>        SUMX (
>            'Sales',
>            DIVIDE ( 'Sales'[WH_StockQty], RELATED ( 'WH_stock'[EXPAND_WH] ) )
>        ),
>        ALL ( Sales[Date] ),
>        ALL ( 'Shop'[City] )
>    )
>VAR SR =
>    CALCULATE (
>        SUMX (
>            'Sales',
>            DIVIDE ( 'Sales'[SR_StockQty], RELATED ( 'SR_stock'[EXPAND_SR] ) )
>        ),
>        ALL ( Sales[Date] ),
>        ALL ( 'Shop'[City] )
>    )
>VAR AL = WH + SR
>VAR TF =
>    IF ( HASONEVALUE ( 'MODE'[MODE] ), VALUES ( 'MODE'[MODE] ), 0 )
>RETURN
>    SWITCH ( TRUE (), TF = "WH", WH, TF = "SR", SR, TF = "ALL", AL )
>```

经过这样处理，库存数据得以正确计算：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191127161720200.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

## 其他补充
如果存在一些产品有库存而无销量，那么以sales表作为主表就会出现部分产品不会被显示，然而类似于本案例，某些时候我们不得不以销售表作为主表，这种情况下，在获取数据时可以使用SQL的Union或M语句Table.Combine解决，但在能用SQL的情况下尽量用SQL，理论上它具有更好的性能。
如果有其他更好的办法，欢迎留言或私信:)







