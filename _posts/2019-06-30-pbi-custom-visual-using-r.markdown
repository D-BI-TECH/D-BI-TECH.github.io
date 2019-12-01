---
layout: post
title:  Power BI 复合饼图
date:   2019-06-30 06:03:50 +0000
image:  04.jpg
tags:   [Power BI,可视化]
---

近期自己尝试开发了一个自定义可视化 - 复合饼图。这款原本在Excel的可视化终于可以搬到Power BI使用了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112920135369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*本可视化仅用于学习目的，其使用案例可[点此](https://pan.baidu.com/s/1OjIXL3FGYmSWJyRELKyg8Q)到达百度网盘，提取码为ppll*

该可视化空间使用R语言开发，我原本打算开发好可以给大家使用，但后来没用解决中文乱码的问题(包括其他开发者发布的使用R语言开发的自定义可视化，我查看过源码，同样没有解决这个问题)，因此就当作是一次试验项目算了，下文将简单介绍一下该自定义可视化。

一、基本介绍
----

如上图所示，这其实是一个很简洁质朴的复合饼图，用法也很简单。该饼图接受两个字段：分类字段和数值字段，缺少一个就会出现警告。  
其原理是，可视化会计算不同标签数据在饼图的占比，占比小于一定阈值的数据会被分离出来，放入到子饼图中显示，主饼图会自动旋转一定角度使之占比较小的部分始终朝向子饼图，同时虚线的位置也会随主饼图的开口角度的变化而变化：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019112920141534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

二、功能
-----

该可视化提供以下功能：

1.中英文语言识别
可视化控件会依据你的Desktop版本显示相应的语言，仅提供英文、简体中文和繁体中文，如果你的版本是英文的它会像如下显示全英界面。

2.设置标签内容
可以设置饼图标签所显示的内容，如果选择"全部显示"则为“标签，数值，百分比”全部显示，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129201438795.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

3.控制标签位置
"外部"意为把所有标签置于饼图外部，"内部"相反，默认为自动模式。

4.设置文字颜色 

5.设置主题颜色
设置饼图边框，虚线以及图例边框的颜色。

6.设置字体。

7.合并为Others
开启"合并较小值"可以将所有因比重较小而被划分至子饼图的分类在主图中合并为"Others"以代表"其他"。

8.显示&隐藏虚线：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129201953474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

9.设置是否显示图例：如下图
10.设置是否显示图例边框：如下图
11.设定图例(文本)大小：如下图
12.设置图例方位：如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129201519491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*（注：点击任意一个图例可以实现在图中排除该项的效果，再次双击可以恢复为显示全部）*

13.调换饼图位置
选择反向可以调换主图与子图的位置

14.设置为圆环图并设定空心半径：
在默认情况下，无论主饼图还是子饼图都显示为实心饼图，但你可以在此处通过改变内圆环半径来使它们变成符合您需求的圆环图。

15.打开工具栏
由于本可视化基于Plotly库开发，因此你可以打开工具栏尝试使用Plotly自带工具(如下图所示套索工具)，在默认情况下工具栏是隐藏的。

16.渐变色

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129201549519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

17.分离Others扇面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129201558150.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

18.错误警报
出现以下情况会显示对应错误警告：
   1. 字段不全
   2. 字段类型不符合要求
   3. 数据全为0或全为Null

*本文原发表于知乎，迁移至本站时省略了一些截图，详细内容可[点此](https://zhuanlan.zhihu.com/p/71617948)到达原文*