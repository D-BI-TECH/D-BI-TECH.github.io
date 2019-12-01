---
layout: post
title:  使用Charticulator自制可视化控件
date:   2019-06-02 06:03:50 +0000
image:  04.jpg
tags:   [Power BI,可视化]
---

<small>*前述：在报表项目中，开发出符合特定需求的可视化控件是十分必要的，但对于大多数Power BI用户来讲，没有编程背景，开发的过程将是耗时而痛苦的。 Charticulator[见尾注]自去年八月发布以来，广受好评，它允许没有编程背景的人，在短短两分钟内开发出一个自定义视觉控件，并且可以导出到Power BI使用(正如开发者所说："Charticulator"是为缺乏编程技能的人设计的，其中可能包括设计师、记者和分析师)，也可以支持数据筛选。然而，国内用户中了解Charticulator的人并不多，但这确实是一个很值得使用的可视化工具*<small>

一、界面介绍
-----
点击此链接以启动Charticulator，你将会看到如下界面：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173239280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

点击左上角的图标打开菜单栏显示如下界面：  
*（警告：导入的数据不要太大，否则你将有机会体验到崩溃的感觉）*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173254720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

单看文字描述当然不好理解，看了下面的教程你将很快清楚它的使用方式。

二、关于使用教程
-----

本文不另行制作教程，只做一些说明补充，因为官方的教程已经足够好，而且我也很推荐看官方的教程，[点此](https://charticulator.com/docs/getting-started.html)到达教程文档页面。
也可[点此](https://charticulator.azureedge.net/videos/gallery/co2_emission_ranking.mp4)到达教程视频，看用户如何短短两分钟内完成精美的数据可视化。
​
我跟随教程，模仿了一个自定义可视化：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173306691.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*（注：我在后续对该图又进行了修改，所以在下文你将看到不一样的可视化）*

其中的国旗(及区旗)部分按如下设置：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173316189.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

最后就是导出为pbiviz格式的PowerBI可视化文件格式，点击导出后，浏览器将会自动下载该pbiviz文件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173325137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

回到PowerBI导入该可视化控件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173332335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

然后。。。什么鬼。。怎么会这样(’-‘)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173359771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*（注：经测试，有的可视化不会出现该问题，但有的确确实实会出现这样的错误)*

除了上图所看到的问题之外，就是自定义可视化的图标能不能换成我们想要的图片呢？下文将会介绍如何debug和修改图标。

三、解决导入控件后出现的问题及修改控件图标
-----

首先，备份该pbiviz文件，然后修改其文件后缀为.zip，解压文件。然后你将会看到一个名为resources的文件夹和一个名为package.json的文件，打开resources，再用文本编辑器打开里面的json文件，映入眼帘的是一大堆代码：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173346988.png)

我在上图标识了一些比较重要的属性，其中绿色框中的displayname是可视化控件在PowerBI的字段设置处所显示的名称。蓝色框的Kind属性可以指定对应字段的类型，黄色框的conditions属性可以指定对应字段放入字段的数量限制，红色框为设置字段接收数据点的数量限制，现在我们仔细往下看就看到问题了，"年份"和"排名"这两个字段的name属性竟是相同的，这其实是不允许的(想不到Charticulator会有这样的bug，希望以后能够解决)，修改其中一个字段属性的name即可(注意有多处), 然后我将开始修改图标，把代码文件拉到最下面，你将会发现该图标的图像编码：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173410238.png)

现在我们需要准备好自己的图像，然后把它转为BASE64编码格式，需要注意的是，在准备图像时，你需要把图像的尺寸强行修改为20*20大小，且需保存为png格式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173418160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

最后，把图像转为BASE64格式(你可以在网上找到很多免费的图像转码工具或网站),将代码粘贴到该json文件中以替换掉原来的图像编码。
最后，保存文件，将原来解压后的文件重新压缩回.pbivz格式，再次导入到PowerBI中，检查下问题有没有解决：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129173505413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

看到上面的图，好消息是：可视化图标成功被替换了，其次年份字段可以正常拖入了，可视化效果也比较接近右边的最终结果了，并且字段属性的显示名称也成功改变了，但坏消息是，为什么这个排序反过来了，而且其宽度为何这么窄，肯定是我在debug的过程中有没有留意到的地方。但回去找找改改一定能解决问题。今天是6月2日号称成人儿童节，给自己多留点时间娱乐吧，买的游戏到了，需要赶紧体验体验。

<small>*尾注：Charticulator是由Microsoft Research团队开发的一个项目，除了六名团队成员，还有一名来自加州大学圣巴巴拉分校的实习生。他们基于Conceptual框架, 使用了包括typescript、react和webassembly在内的技术, 将Charticulator作为一个HTML5应用程序实现*<small>

*End~*
