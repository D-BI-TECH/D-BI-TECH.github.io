---
layout: post
title:  PowerApps助力PowerBI实现数据写回
date:   2019-08-01 06:03:50 +0000
image:  08.jpg
tags:   [Power BI,PowerApps,Office 365]
---

*注：本文旨在介绍Power BI如何利用PowerApps实现用户在前端对数据源进行增删查改，关于此，你也可以在Google上找到更详细但较零散的资料*

#### 正文

在SSAS多维数据集中，开发者可以给数据开启"写回"功能，以便用户可以在前端自行修改某些数据，进行What-If分析。然而在Power BI中，用户只能查看数据或通过参数修改度量值的公式，却不能影响到后端数据源的数据，这是很遗憾的。假如有这样的一个情况，我们有一个会员表，有些会员的信息，系统无法录入，只有相关业务人员了解实际情况，又比如说一些由业务人员本身所确定的数据，比如选品、产品折扣等，对于这些数据，如果他们能自己直接在报表前端追加它们，把这些数据写到数据源，然后再通过报表反映这些新的数据，那将明显节约时间和沟通成本。而这些操作，PowerApps可以帮你实现。

*注：PowerApps是一个与Power BI并列的微软Power平台三大应用之一，它可以与基于云端的Power BI协同工作，关于此，在我[此前的一篇博客]({{site.baseurl}}/microsoft-flow-for-pbi/)有更多介绍*

其大体原理是，PowerApps作为一个用于构建应用的工具，它本身可以连接到数据源（Excel、OneDrive、SQL Server等），并且提供丰富的方法，其中就包括修改、增加和删除数据源的数据。在PowerApps构建好一个项目后，回到Power BI调用该可视化插件，就可以在Power BI中使用你在PowerApps中构建的应用了，因此借助PowerApps, 用户就可以在Power BI报表中进行数据写回了。

首先进入PowerApps并登录，你可以新建一个环境或使用默认预设的环境（和Power BI Pro一样，它需要按月付费，因此你可以选择试用30天（实际可以有60天）），紧接着一个重要的环节是连接到数据源，如下图所示:

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120117282525.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*注：如果你使用类似本文的本地或远程服务器的数据源，由于它是非云端的，因此你必须事先安装和配置好网关， 注意，个人版网关在此处是不能用的*

点击[建立]，可以看到PowerApps提供丰富的模板，本文选择第四个---从资料开始，以便在开始项目之前预先连接好数据源。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201172842633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

进入PowerApps Studio界面后，插入表单组件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201172856698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

插入该组件后，给该组件配置好数据源和引用字段(注:所引用的表需要有主键)：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201172904816.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

得到如下输入框：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120117291922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

设置该控件的Item属性：

>```Python
Item = BrowseGallery1.Selected
>```

使其得以引用上一个控件(黄色标识)所被选择的值。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201172928917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

新增一个按钮，命名为"提交", 在其OnSelect属性中调用.SubmitForm方法，该函数即用于将其所引用的数据提交给所连接的数据库：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201172937608.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

然后，点击右上角三角形运行该项目(或按住Alt)，在输入框中输入数据（今天是建军节，因此我输入了伟大的朱总司令的姓名）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201172952516.png)

点击提交后，回到数据库或在Power BI中刷新数据，你就可以看到我们所输入的数据已经写回到了数据源（选中它你也可以修改数据）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120117300269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

你还可以插入更多的控件，并给它们的属性设置函数，比如删除操作，为对应控件的OnSelect属性设置公式为：

>```Python
OnSelect = Remove('[dbo].[testing]',ThisItem)
>```

这样，运行项目后点击该控件就可以实现删除数据源的数据。你也可以使用UpdateIf()函数修改满足指定条件的数据。
PowerApps提供丰富的函数可以供你将应用变得更智能，更加定制化，本案例仅冰山一角，你可以点此参阅官方的函数文档。完成项目后，储存并发布，你可以选择将该项目共享给你的组织成员：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201173014278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

最后，回到Power BI, 导入PowerApps可视化并开启你的应用，你就可以在Power BI上使用你所构建的应用来实现数据的写回操作了。(注：确保你在Power BI登录的账号和PowerApps是同一个)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201173025374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

我在上图Power BI中用该控件将朱德修改为陈毅并追加一行数据"聂荣臻", 如你所见，数据源成功实现了更改和追加，并在Power BI报表刷新后返回了我所输入的新数据:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201173035655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 其他方面

- 数据源表的改变可能不会立刻反映到PowerApps中，不同的数据源可能有所不同。
- 你无需担心该功能实现后的权限控制问题，一是你可以将该应用分享给指定用户，二是即使该应用分享给了全体组织成员，PowerApps内部提供获取用户信息的函数可以帮助你控制应用中各个功能的使用权限，三是你可以在网关配置上设置权限类型。
- PowerApps无法在Power BI Report Server使用，你可以在Desktop调试，但你只有发布到Power BI Service才是有效的。