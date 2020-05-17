---
layout: post
title:  Power BI：用DAX实现动态R-Visual
date:   2020-05-17 03:03:50 +0000
image:  08.jpg
tags:   [Power BI,R,可视化]
author-name: Davis ZHANG
author-image: Davis.jpgs
level: 进阶
---

**(本文将会讲解利用DAX度量值驱动R可视化的方法, 将会同时涉及DAX以及R语言相关知识)**

### 前述

微软Power BI自2015年发布以来, 随着其Desktop软件每个月的频繁更新以及AppSource第三方可视化控件的不断完善, 其可视化能力已今非昔比, 然而用户的需求总是更高一丈, 比如, 如何才能在图表中绘制自定义标记, 辅助线或实现一些较为不规则的显示效果呢? 显然, 这种需求的自定义化的程度太高, 微软不可能为每一种自定义效果都开发一套可视化, 因此在2017年一个十分明智的方案: R Visual 被加入到了Power BI, 这允许用户能够在Power BI使用R语言进行绘图, 尤其是R语言包含了多种受支持且功能强大的开源包(如ggplot2), 因此只要用户拥有一定的R编程基础,就能在Power BI绘制出任何类型的高度定制化图表(随后,微软还为Power BI加入了Python可视化) 然而可惜的是, 利用该功能的用户并不多, 一方面由于R视觉对象本身的局限性, 比如不支持工具提示, 在发布到Web的报表以及本地部署的报表中不受支持, 另一方面则是多数Power BI用户对该功能缺乏了解, 或是低估了R可视化赋予报表的强大能力. 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517220956570.png)

### 效果

本文将以此前发布的文章[《Power BI: 关于钻取, 书签与按键》](https://d-bi.gitee.io/pbi-drillthrough-bookmark-and-button/)中的蜡烛图为例, 分三大步骤:

1. 实现绘制蜡烛图,网格线以及坐标轴 (R语言熟练者可跳过)
2. 在图中添加自定义辅助线或标记 (R语言熟练者可跳过)
3. 结合DAX, 实现用切片器控制R可视化, 实现深度交互(R+DAX 本文重点)

（注：关于蜡烛图，我开发了专门的可视化控件，详情[点此]({{site.baseurl}}/docs-pbi-visual-candlestick/)）

效果参考下图: 

- 标红部分, 实现在切片器选择不同的颜色标准来切换蜡烛图的颜色, 中国标准为红涨绿跌, 国际标准与之相反. 
- 标黄部分, 实现可以控制是否显示日均线以及日均线的切换(图中黄色曲线)
- 标蓝部分, 此部分可以实现让用户控制是否显示股市熔断的标记以及定点分析(此处还实现了判断所选股市是否有熔断机制,如果没有则不显示的效果), 其实现原理与前两个步骤相通, 本文不再赘述, 留读者自行发挥

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517182820913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 1.绘制蜡烛图, 网格线及坐标轴

选中R Visual后, 拖入必要的字段: 日期, 最高价, 最低价, 收盘以及开盘价, 这些字段会在R执行器中使用data.frame的方法生成新的数据集.
然后输入以下代码:

(注: 本案例中, 五个字段名分别为: 日期: date  收盘价: CloseIndex  开盘价: OpenIndex 最高价: HighIndex  最低价: LowIndex 实际还有第六个字段: index_code, 这是指数代码, 在此是为了实现当用户选则不同的大盘指数时, 使图表可以使用对应的指数数据集,如果你的数据集里只有一种指数则不需要此字段. 以下代码,你只需将这六个字段的名称改成你自己的即可直接套用. 另外, 代码中有许多参数可供修改，比如颜色代码, 你可以改成你自己喜欢的颜色)

```SQL
CODE <- dataset$index_code
DATE<-dataset$date

if(length(unique(CODE))>1)
{
    dataset <- subset(dataset,dataset$index_code == CODE[[1]])
} else {
    dataset = dataset
}

D<-dataset$CloseIndex - dataset$OpenIndex
N<-length(dataset$date)
M <- N-11
w<-0.3
xIndex <- round(seq(from=1,to=N, na.rm=T,length=5))
xText <-substr(dataset$date[xIndex],1,10)
yIndex <- seq(from=min(dataset$LowIndex, na.rm=T),to=max(dataset$HighIndex, na.rm=T),length=7)
yText <-round(yIndex,digits = 2)

left_margain <- if(max(nchar(yText))-3>0){0.5*max(nchar(yText)-3)+2} else {2}
bottom_margain <- 1.8
box_color = "grey"
par(mar=c(bottom_margain,left_margain,1,0.5)+0.5,family='serif',bg="transparent")

p <- plot(c(1:N),dataset$CloseIndex,type = 'n',xaxt = 'n',yaxt='n', xlab = '', font.axis = 1.5,bty="n",ann=FALSE)+
abline(h = yText, lty = 2, col = "#FF2435")+
abline(v = xIndex, lty = 2, col = '#FF2435')+
 for(i in 1:N)
 {
 lines(c(i,i),c(dataset$LowIndex[i],dataset$HighIndex[i]),col = box_color,lwd = 1)
 x<-c(i-w,i-w,i+w,i+w)
 y<-c(dataset$OpenIndex[i],dataset$CloseIndex[i],dataset$CloseIndex[i],dataset$OpenIndex[i])
 if(D[i]<0)
 {
  polygon(x,y,col='#399599',border=box_color)
 } else {
  polygon(x,y,col='#FD625E',border=box_color)
 }
 } + 
 axis(side=1,at=xIndex,labels=xText,col.axis='#FF2435',tick=FALSE,las=1) + 
 axis(side=2,at=yIndex,labels=yText,col.axis='#FF2435',tick=FALSE,las=1)

p

```

效果如下:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517191752578.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 2. 添加自定义辅助线(或标记)

此步骤是R Visual的核心价值, 它展现出R代码的力量, 可以实现其他可视化控件无法做到的高度定制. 此处的需求是添加一条最新价格标记线, 这里只需在以上代码基础上, 增加两部分, 分别绘制辅助线和轴标签:

- 辅助线:

```SQL
abline(h = dataset$CloseIndex[N], lty = 2, col = 'SteelBlue')
```

- 轴标签:

```SQL
axis(side=2,at=dataset$CloseIndex[N],labels=round(dataset$CloseIndex[N],digits=2),
	col.axis='SteelBlue',col.lab='green',tick=FALSE,las=1) 
```

效果如下:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517201821937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 3. 利用DAX建立度量值控制R可视化

##### 基本原理案例: 日均线部分

此步骤首先以添加日均线为例, 最终需要实现用户可以控制图表是否显示日均线, 以及切换日均线 (5日均线, 10日均线等). 首先, 利用DAX计算股票价格(收盘价)的移动平均值 (DAX动态移动平均计算以及实现用户控制可视化显示可[参考此文](https://d-bi.gitee.io/enhance-Chart-Interactivity/))

```SQL
MA_Value = 
VAR SELECTED =
    IF ( ISFILTERED ( PARA_MA[MA DAYS] ), MAX ( PARA_MA[DAYS] ), FALSE () )
VAR _MA = 
    SWITCH (
        FALSE (),
        BLANK(),
        SELECTED, CALCULATE (
            AVERAGEX (
                FACT_STOCK,
                CALCULATE ( AVERAGE(FACT_STOCK[close]), ALLEXCEPT ( FACT_STOCK, 'FACT_STOCK'[date],FACT_STOCK[index_code] ) )
            ),
            DATESINPERIOD (
                'FACT_STOCK'[date],
                LASTDATE ( FACT_STOCK[date] ),
                SELECTED*-1,
                DAY
            )
        )
    )
RETURN
IF(ISFILTERED(PARA_SHOW_MA[NAME]),_MA,BLANK())
```

移动平均度量值建立完成后, 将其拖入R可视化, 然后在上一步骤的R代码基础上增加以下代码:

```SQL
lines(c(1:N),dataset$MA_Value,col="yellow",type='l',lwd=0.5)
```

如下, 日均线成功添加, 不仅如此, 还可以通过切片器切换显示不同的日均线, 这里的一个重要原理是切片器通过传参的方式将值传递给了度量值"MA_Value", 度量值又将值传递给R Visual最后出图(关于DAX部分还是推荐[参考此文](https://d-bi.gitee.io/enhance-Chart-Interactivity/)): 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517203612584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

同时, 还实现了用户控制显示或隐藏日均线的效果, 如下图所示, 取消勾选"显示日均线"以实现隐藏效果: 

(注: 此处利用公式中:IF(ISFILTERED(PARA_SHOW_MA[NAME]),_MA,BLANK())实现)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517204720770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

##### 进阶案例: 切换颜色规则案例

在日均线案例中, 我们是利用由DAX建立的度量值的数值发生的变化, 从而使传递至R可视化的值也发生变化, 进而实现切片器对R可视化的交互, 但此处我们需要切换此蜡烛图的颜色规则, 这即使是在部分默认甚至多数第三方可视化里, 都无法做到让前端用户去控制图表的颜色, 而DAX要如何将对应的颜色值, 从用户手里传达给R呢? 

实现此效果的方法, 是新建一个专用于判断用户选择的度量值, 此处, 如果用户选择中国标准, 返回参数1, 如果是国际标准则返回参数-1, 然后将该度量值导入R可视化, 在R代码中写IF判断, 如果该度量值为正则红涨绿跌, 否则绿涨红跌. 依照这个思路, 不难得出:

```SQL
ColorForRVisual_STOCK = 
VAR _R = IF(
	HASONEVALUE(PARA_COLORCONTROL[RULE]),VALUES(PARA_COLORCONTROL[VALUE]),BLANK()
	)
RETURN 
IF(_R,1,-1)
```

接下来, 用户就可以用以下切片器控制颜色规则了:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517210307533.png)

然而, 事实不像想象般顺利, 因为度量值向R传递的单个值:1和-1, 是无法在R中与其他字段组成正确的数据集的, 因为单个值不含上下文, 在R组建数据框时会出错. 因此, 解决问题的真正办法是通过与字段值进行计算来强制为度量值添加上下文:

```SQL
ColorForRVisual_STOCK = 
VAR _R = 
IF(
	HASONEVALUE(PARA_COLORCONTROL[RULE]),VALUES(PARA_COLORCONTROL[VALUE]),BLANK()
	)
RETURN 
IF(_R,1,-1)*'FACT_STOCK'[CloseIndex]
```

最后, 在目前R代码中, 将以下部分:

```SQL
 if(D[i]<0)
 {
  polygon(x,y,col='#399599',border=box_color)
 } else {
  polygon(x,y,col='#FD625E',border=box_color)
 }
```

改为:

```SQL
 if(dataset$ColorForRVisual_STOCK>0){
 	if(D[i]<0)
	 {
  	polygon(x,y,col='#399599',border=box_color)
 	} else {
  	polygon(x,y,col='#FD625E',border=box_color)
	 }
 } else{
	 if(D[i]>=0)
 	{
  	polygon(x,y,col='#399599',border=box_color)
 	} else {
  	polygon(x,y,col='#FD625E',border=box_color)
 	}   
 }
```

这样就成功实现颜色规则应用了。如下所示，切换颜色规则前：

![在这里插入图片描述](https://img-blog.csdnimg.cn/202005172115395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

切换颜色规则后:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517211652385.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 总结

本文展示了R可视化强大的可定制性以及交互能力, 并说明了技术细节, 事实上, 依据本文的原理, 能够实现的效果远不止于此, 如果你是一名熟悉R语言语法的Power BI用户, 那么本文已然为你打开了新世界的大门，待你进一步探索. 
 
(本文涉及的可视化在此前已由本人分享至[Power BI Community](https://community.powerbi.com/), 你可以在社区下载完整的R代码([点此到达](https://community.powerbi.com/t5/R-Script-Showcase/SIMPLE-K-CHART-Candlestick/td-p/1013295)) )
