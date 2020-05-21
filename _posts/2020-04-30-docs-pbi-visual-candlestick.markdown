---
layout: post
title:  Power BI Visuals - Candlestick 介绍文档 
date:   2020-04-30 08:03:50 +0000
image:  06.jpg
tags:   [Power BI,Power BI Visuals,可视化]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 无
---

**(注:本文将简要介绍新的Power BI可视化--Candlestick的使用方法.)**

### 简介

Candlestick是由本人(Davis ZHANG)使用R语言开发的Power BI可视化, 它将在数日之后(取决于审核的进度)发布在[Microsoft AppSource](https://appsource.microsoft.com/en-us/marketplace/apps)并提供给大家下载使用. 该可视化--Candlestick,顾名思义即为用于分析股票市场的蜡烛图(K线图), 开发该可视化的动机是近期因疫情造成的股市下跌提高了人们对金融市场的关注度,而目前在可视化市场中却没有专用于分析金融股市的可视化, 唯一的K线图是由OKViz发布的可视化--[Candlestick by OKViz](https://appsource.microsoft.com/en-us/product/power-bi-visuals/WA104380952?tab=Overview)
, 但该可视化存在一些局限性, 因为它是按照一般可视化的思路设计的, 因此也不会考虑到因休市日产生的数据断点的问题(截至到本文发布之时), 另一方面,我在去年也曾有开发PowerBI可视化的经历, 多少积累了一点经验, 开发一个新的可视化应该不会耗费太多时间成本, 因此Candlestick在这个想法之下诞生.


### 设计思路

作为金融类可视化的最初版本,我想尽可能简单化,提供基本的蜡烛图以及日均线的展示. 最初的设计思路是为Candlestick开发两个版本, 一个开发版,一个用户版, 开发版即传统的Power BI可视化设计思路, 它将可视化呈现的结果的决定权全部交由报表开发者,比如图表的颜色和样式,字体的大小等等,我甚至会加入一些预定义的元素; 而用户版则有很大不同, 开发者只需要将必要的字段拖入, 可能还是需要设置一下颜色字体等基本的设置,但有些设置,比如图表的样式,标注文本等等,这些设置将会以按键或其他的形式在前端界面交由用户决定,这样的话就能够给用户更好的使用体验, 但受限于本人时间精力, 暂时只开发了第一种版本,不过这也并非坏事,这样我可以通过发布第一个版本的可视化获得一些高价值的用户反馈以及一些可能存在的bug, 以便我可以在未来的版本中改进它.


### 关于使用

在Power BI Desktop, 导入该可视化,系统将会提示您安装必要的包(如果你没有安装过的话), 包括GGPlot2等等, 如果是直接在Power BI Service 使用,则无需进行额外的安装操作,因为Power BI Service已经集成了必要的R脚本执行环境, 但如果要将带有该可视化的报表发布到空间, 需要至少拥有Pro Licence([点此](https://docs.microsoft.com/en-us/power-bi/visuals/service-r-visuals#known-limitations)查看目前使用R可视化的已知限制). 

在该版本的Candlestick中(v1.0.0 后期若有改动可能不同), 你只需拖入股票价格数据(开盘,收盘,最高以及最低)即可, 这里需要注意的是, 数据要么全部使用计算列,要么全部使用度量值,如果计算列和度量值混用,则可能造成图表展现错误的问题.


### 功能架构

此处用一张流程图展示最初版本的功能架构:

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020043016223659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 部分截图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200430162654300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200430170504386.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200430171247349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 已知的限制

以下是目前该可视化主要的问题, 如果您发现了其他问题欢迎向本人反馈.

1. 该可视化最多接受10000行的数据,超出的部分会被截断
2. 复制到剪贴板的功能目前限制在1000行数据,超出部分会被截断(此处我会在未来版本根据用户反馈考虑是否增加行数限制),用户依然可以使用可视化默认的导出到csv的选项导出数据,但这会导出很多无关字段,比如颜色值等等.
3. 中文,日文以及韩文字符会出现乱码问题,将在下一个版本中得到修复

