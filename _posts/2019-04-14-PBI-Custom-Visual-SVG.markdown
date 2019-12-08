---
layout: post
title:  Power BI之SVG自定义可视化
date:   2019-04-14 06:03:50 +0000
image:  12.jpg
tags:   [Power BI,可视化]
author-name: Davis ZHANG
author-image: Davis.jpg
---

一、简述
-----
业务人员常常希望以可视化的方式对数据进行切片，然而PowerBI自带的可视化控件往往难以满足需求。比如当你要分析电影院各个座位的销售数据时，普通的柱状图难以体现出相邻座位之间的数据关系，你更需要的是用影院座位图直接展示出数据，并点击不同的座位做切片。或者，你希望有一张上海区划图来研究上海市各区的房价。类似于如下的效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129112616807.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

数据展示的是2018年广州和深圳的街道办总数。点击任意城市以显示对应数据
事实上，实现这个效果，只需要简单几步。

二、基本过程
-----
首先进入该网站：[SYNOPTIC DESIGNER FOR POWER BI](https://synoptic.design/)

你可以把自己的图片拖入编辑器中，也可以导入网站提供的图形样板：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129112319324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

编辑完成后，点击"EXPORT TO POWER BI"，下载后得到一个SVG图像文件。用文本编辑器打开它：
(SVG是一种矢量图形格式，它有两个重要特性：1.任意放大收缩而不会影响画质 2.其图像可以通过文本编辑器进行编辑)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129112759150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

如果打开SVG时全部挤成一行，可以在[XML Beautifier](http://xmlbeautifier.com/)进行快速排版。  
蓝色方框内的id就是代表每个店铺的图形，为了让点击每个图形时显示对应店铺的数据，或者让各个店铺图形的颜色深浅来表示数据大小，就需要让SVG里面的id和主表数据的id进行匹配。你可以尝试在Excel用randbetween()生成一个测试表并导入到PowerBI。  
然后，导入可视化控件Synoptic Panel,再导入刚刚下载的SVG图片：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129112931958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这样就完成了自定义图像切片：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129112953656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

此外，该控件还可以支持多个SVG图像的切换，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129113023402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

三、使用远程链接访问SVG图片
-----
除了从本地导入SVG，我们也可以使用远程链接来访问SVG，特别是当报表需要多人共享时。出于本案例需要，我在Github上传了我此前下载过的世界主要国家或地区的地图，以及自制的广东省地图共30张，你可以点击这里下载它们或复制链接。
更简单地，你可以点此下载本案例Excel表格，然后复制如下代码到你的表格来创建计算列以实现这种效果：

>```Python
Map URL = 
"https://raw.githubusercontent.com/hdiiasa/SvgMap-for-PowerBI/master/"&
SUBSTITUTE('Test'[Country]," ","")&
".svg"
--使用SUBSTITUTE是因为部分国家或地区的全称含有空格，而网址中的全称需要避免空格转换为“20%”
>```

四、SVG图像制作迷你图
-----

将SVG语法嵌入到DAX中以实现mini图效果，例如以下公式：

>```Python
SalesTrend = 
var
groupby = if(HASONEVALUE(groupbyoption[GroupBy]),values(groupbyoption[GroupBy]),0)
var
ShopNo = if(HASONEVALUE('SKPI_Company_Shop'[No_]),values(SKPI_Company_Shop[No_]),0)
var vt = SUMMARIZE(all('Date'[Date]),'Date'[Date],"a",Salesman[SalesQty])
var max_value = 
maxx(vt,[a])
var
Interval = 1
var
w = max_value/4
var
lastday = LASTDATE(Salesman[Date])
var ymaxvalue = max(Salesman[Qty])
var
v0 = max_value-CALCULATE(sum(Salesman[Qty]),dateadd('Date'[Date],-1,DAY))
var
v1 = max_value-CALCULATE(sum(Salesman[Qty]),dateadd('Date'[Date],-2,DAY))
var
v2 = max_value-CALCULATE(sum(Salesman[Qty]),dateadd('Date'[Date],-3,DAY))
var
v3 = sumx('Salesman','Salesman'[SalesQty])
RETURN
"data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' x='0px' y='0px' viewBox='0 0 "&max_value&","&max_value&"'><rect x='"&w&"' y='"&v0&"' width='"&w&"' height='"&max_value&"' style='fill:%2301B8AA;stroke-width:2;stroke:white' /><rect x='"&w*2&"' y='"&v1&"' width='"&w&"' height='"&max_value&"' style='fill:%3351A8AA;stroke-width:2;stroke:white' /><rect x='"&w*3&"' y='"&v2&"' width='"&w&"' height='"&max_value&"' style='fill:%3351A8AA;stroke-width:2;stroke:white' /></svg>"
>```

*注：关于迷你图制作，需要对SVG编码有一定了解，[点此](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Tutorial)查阅相关教程*

效果参考下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129114805660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

​*注：根据博主当时的测试，mini图不能在本地部署的PBI Report Server正常显示*
