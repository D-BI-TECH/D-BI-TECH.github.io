---
layout: post
title:  PBI Report Server报表页面自动刷新
date:   2019-11-26 06:03:50 +0000
image:  02.jpg
tags:   [PBI Report Server,SSRS]
author-name: Davis ZHANG
author-image: Davis.jpg
---

在Power BI，一个很尴尬的情况是，无论是在Power BI Services 还是PBI Report Server，就算你为你的报表设置了高频率的数据刷新计划甚至是使用DirectQuery，报表页面都无法即时显示最新的数据，因为报表页面所展示的是你在打开报表页面前的最新数据的缓存结果，而后续的数据更新，只有当你手动刷新页面或重新打开报表才会返回最新的结果。首先，作为一名BI，“手动”这个词是很不讨人喜欢的，更重要的是，在生产环境中，管理者需要报表能够显示在大屏幕上，无需人为干预就能自动展示最新的数据（比如流水线各个环节的即时情况，机器状态等等）。关于此，一个好消息是在Power BI Desktop 10月的更新文档中，引入了页面刷新的预览功能，只要报表数据源是DirectQuery模式，就可以实现报表页面的自动刷新：

![图片来自Microsoft文档](https://img-blog.csdnimg.cn/20191201185229241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

但一个坏消息是，本地部署的Power BI Report Server这次依然没有跟上时代步伐，微软没有为PBI Report Server提供上述功能，至于未来会不会提供，我只想说，R可视化功能在将近四年前被加入到Desktop时Power BI Services就已经能够支持了，而PBI Report Server至今无法使用R Visual。
但还好，在Power BI报表服务器实现自动页面刷新，我此前研究了一下，也是有办法实现的，而且测试效果还不错，且能够轻松配置。

先介绍一下Google到的其他方案，即是在浏览器上安装定时刷新页面的插件，以Chrome为例，你可以添加一个名为"Auto Refresh"的插件，然后你可以设置页面刷新间隔：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201185301404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这确实可以解决Power BI报表在报表服务器WEB门户的自动页面刷新的问题，但这种方式的效果却不太能让人满意。当你用这种方式刷新页面时，整个页面需要重新加载，整个报表大屏幕会在每次刷新时出现短暂的白页，如果你是一小时刷新一次倒没什么，若是不到一分钟就刷新一次，那体验就相当差了。
我随后想到的方法是在HTML上设置刷新参数，然后把我们需要自动刷新页面的报表用iframe嵌入到HTML中，这样我们可以很轻松的实现页面局部刷新，让它只刷新报表本身，测试的结果是，它提高了每次刷新页面加载速度，但还是无法避免白页停留。所以我们真正希望的效果不应该是刷新页面，而是让报表后台重新执行一次计算返回最新的结果，就像我们点击"刷新"按键一样：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120118530641.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

可能有人已经想到了一个办法，就是设置一个定时脚本去点击这个"刷新"按键，但更好的方法是找到该按键的元素，在浏览器后台写一个简单的javascript去执行它。首先在WEB门户打开报表然后右键查看元素，我们能找到"刷新"按键对应的元素id：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201185317355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

但同时存在一个问题，该元素id与"刷新"按键并非一对一关系，当你执行该元素时，它执行的实际上是"在Power BI Desktop"中编辑。因此你需要修改它的id，比如在本案例中我将其修改为"c"。
最后，打开控制台，输入以下js代码(没想到只要一行就搞定了)：

>```Javascript
setInterval(function(){document.getElementById("c").click(); }, 5000);
>```

其中"5000"指5000毫秒，即命令浏览器每隔5秒执行一次"刷新"，然后执行该命令，你会发现你的报表每隔5秒就会自动刷新数据了，并且和你点击"刷新"按键的效果一样，完全不需要重新加载页面。并且该方法适用于任何类型数据源的Power BI报表，而不仅仅是Direct Query。

*如果喜欢本文，点击下方logo分享或将链接分享给你的朋友或同事吧~*  
*End~*