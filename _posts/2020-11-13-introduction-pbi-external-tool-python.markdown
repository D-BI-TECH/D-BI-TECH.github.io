---
layout: post
title:  Power BI External Tool 开发指引
date:   2020-11-13 01:03:50 +0000
image:  19.jpg
tags:   [Power BI,Python]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*这篇文章将会讲解使用Python开发Power BI External Tool的大体实现过程*

### 前述

上个月，鉴于我曾经发布了[DAX Beautifier](https://github.com/D-BI-TECH/dax-beautifier), Michael Carlo建议我分享一篇关于构建Power BI External Tool的博客，我很快答应了他，但由于我当时还在度假，并且在度假回来后，由于工作上有一些急待处理的事情而经历了繁忙的一周，因此这篇博客就一直拖到现在。在这篇博客里，我会介绍它的开发过程，包括coding，打包为exe以及构建安装程序，同时，我会尽可能减少文章的篇幅，为读者保留一点探索的空间，因为“不完美才是真正的完美”，最后，我系望这篇文章会对Power BI社区的开发者提供有用的参考。

### 开发

在这一段落里，我将以我过去发布的DAX Beautifier为例，讲解大致的开发过程。

> 尽管一门语言有时候可以搞定一切，但用适合的语言解决合适的问题才是效率最高的。

首先你必须清楚你究竟要开发一个什么类型的工具？然后你就能大致决定使用何种开发环境或者语言。Tabular Editor是目前最完美的表格模型读写工具，他已经包含了你想要的大部分功能，比如新建度量值组，模型层翻译，度量值管理等等，它使用.net开发，优点是速度快，体积小，集成度更高，但如果你的工具需要完成一些数据处理的工作，也许Python是更明智的选择，我们知道Python是解释型的高级语言，它有更丰富的Library，以及你无需在开发过程中完成编译（但也正因为此，打包后的应用程序往往体积较大，这会在下文说明），对于DAX Beautifier这种轻量级工具而言，使用Python是合适的，何况我们还有[pythonnet](https://pypi.org/project/pythonnet/)这个强大的package，这使得我们可以很轻松地利用Python和TOM模型进行交互。

接下来，是时候为你的应用程序理清思路了。 绘制流程图是一个好办法。对于DAX Beautifier而言，逻辑和过程都十分简单，获取模型对象后，将需要格式化的度量值发送到DAX Formatter，然后将结果写回到表格模型中，如下：

![flow](https://img-blog.csdnimg.cn/20201111152040248.png#pic_left)

我们必须明确程序执行的逻辑，并确保高效率。有人可能会问，为什么不使用Python去构造格式化的逻辑，而是调用DAX Formater提供的API呢？因为这样并不明智，而且会拖慢工具的执行速度以及代码的维护成本，即使像SQLBI这样在DAX领域顶尖的团队，也不能完全避免DAX在格式化时出现的BUG，而自行构造格式化逻辑，不仅需要对DAX有深入的理解，还意味着大量的试错成本，而利用现成的API，则可以保持项目的轻量化，也能提高工具的使用效率。

*关于使用Python来开发Power BI External Tool，你还可以阅读David Eldersveld的[此篇文章](https://dataveld.com/2020/07/20/python-as-an-external-tool-for-power-bi-desktop-part-1/)*

### 打包

这个段落里，将会告诉你如何将一个开发完成的py程序打包成exe。你可以[点此](https://github.com/D-BI-TECH/dax-beautifier/blob/master/dax-beautifier.py)下载DAX Beautifier的源代码以便完成后续测试。

事实上，你完全不需要将py打包成exe就可以在PBID使用你所开发的工具，但我们必须考虑到使用者的感受，有许多Power BI开发者来自Excel社区，他们很可能对编程一无所知，他们也不希望把时间浪费在安装各种繁琐的python运行库上，所幸的是，我们有[pyinstaller](https://pypi.org/project/pyinstaller/)这个十分便利的package，使我们可以轻松地为我们的应用程序创造编译环境。

使用Pyinstaller非常简单，安装Pyinstaller后,  你可以使用CMD导航到.py文件目录下（如果你安装了Anaconda, 我推荐你使用Anaconda Prompt）:

```SQL
cd <your .py file directory>
```

通常，运行以下代码即可。

```SQL
pyinstaller -F -w <your .py file name>
```

但如果你也使用了第三方库（比如pythonnet），你必须引入第三方库的存放路径。

```SQL
pyinstaller -p <your packages directory> -F -w <your .py file name>
```

运行命令后，如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201112111344172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

打包成功后，你可以在dist文件夹下找到你所创建的exe文件。

### 最后一步

在完成.exe程序后，事情还没有结束，因为我们还需要创建一个安装程序，使得你所开发的工具可以自动地加载到Power BI Desktop中。我推荐你使用NSIS来完成安装程序的创建，你需要分别下载和安装[NSIS](https://nsis.sourceforge.io/Download)以及其IDE "[HM NIS Edit](https://sourceforge.net/projects/hmne/)"，关于NSIS的使用，可以参考[此文档](https://nsis.sourceforge.io/Simple_tutorials)或[此博客](https://community.broadcom.com/symantecenterprise/communities/community-home/librarydocuments/viewdocument?DocumentKey=b3a0e63a-c009-44aa-8852-4ca89daea50b&CommunityKey=ef59d715-7ea1-41c6-97f3-dd1bcc10d0c3&tab=librarydocuments)。此外，不要忘记在安装程序中将你的pbitool.json文件放置在以下目录：

```SQL
C:\Program Files (x86)\Common Files\Microsoft Shared\Power BI Desktop\External Tools
```


### 更多

以上就是开发Power BI External Tool的完整介绍， 这并不是一件难事，但目前我们只能对模型层进行读写，我们希望未来报表层也能拥有读写能力，这样我们就能在开发思路上有更大的创意空间，如果你也同意这个想法，我非常希望你可以为[此idea](https://ideas.powerbi.com/ideas/idea/?ideaid=4657bb93-bb25-48c2-9304-26bfd8def58c)投票。
