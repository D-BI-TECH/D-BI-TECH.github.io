---
layout: post
title:  PBID外部工具DAX Beautifier更新文档
date:   2020-08-17 01:03:50 +0000
image:  07.jpg
tags:   [Power BI,DAX]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 无
---

*本文是Power BI Desktop外部工具DAX Beautifier的版本更新说明（随版本动态更新），有关DAX Beautifier的详情与使用请参考此Github页面的[Readme](https://github.com/DavisZHANG-BlogOnly/dax-beautifier)（英）或[《PBID外部工具DAX Beautifier使用必读》](https://d-bi.gitee.io/pbi-external-tool-dax-beautifier/)（中），本文不再赘述。*

最初版本号为1.0.0，工具每次修改或移除，追加功能都会增加新的版本号，这些更改主要基于用户的反馈以及本人的测试。如果想使用该工具的历史版本可[点此到达](https://github.com/DavisZHANG-BlogOnly/dax-beautifier/tree/master/previous-versions)下载页面。

### 版本 1.0.1

(发布：2020-8-7)

1. 如果没有检测到需要格式化的公式，程序将会跳过提交环节提前结束。

2. 程序报错时windows窗口将停留1秒钟，避免因闪退导致难以捕捉到错误信息。

3. 追加新功能：**DAX增量格式化**。该功能允许用户仅修改在近N个小时新增或修改的DAX公式。运行程序后将会收到提示信息（如下图所示），如果输入0，意味着格式化PBI文件内全部DAX公式，如果输入其他数字，比如3，意味着仅格式化在当前时间的前3个小时内修改过或新建的DAX公式。

![v:1.0.1](https://img-blog.csdnimg.cn/20200809190938949.png)

该功能在PBI报表开发过程中十分有用，尤其当报表进行版本迭代且涉及的公式较多时。**当你的PBI文件内的DAX公式已经在上一个发布版本完成了格式化，此时你只需要格式化你本次涉及的DAX公式，而无需格式化全部公式**。比如，在你的报表上一个版本包含多达100个DAX公式，并且已经在上次发布前完成了格式化，在当前版本你仅仅修改了一个度量值，新增了一个计算列，使用该功能可以帮助你仅仅格式化当前版本涉及的2个公式而非全部公式，这将大幅节省你的时间。

该功能的另一个好处是使用**近N小时**的判断方法。因为人们在报表开发过程中，多数情况下你不会记得你新增的度量值叫什么名字（因为实际的报表模型可能会很复杂），以及它们在哪个表里，但人类大脑却十分擅长对时间印象的记忆，你能够清楚的知道你在两个小时前修改了一些度量值，但你未必记得你是在S表中修改了一个名为"_this_is_a_measure"的度量值或在P表新增了一个名为"_with_a_long_name"的计算列。在DAX Beautifier，**你需要做的仅仅是输入一个代表小时的阿拉伯数字！**

### 版本 1.0.2

(发布：2020-8-13)

1.更改了DAX Beautifier的安装界面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200817111908273.png)

2.更改了图标大小及其他细节。

### 版本 1.0.3

(发布：2020-11-18)

该版本增加了对Power BI Report Server（PBIRS) 的支持。

由于截至目前，PBIRS暂不支持外部工具，针对于此，DAX Beautifier新增了独立运行模式。

你只需直接双击dax-beautifier.exe程序本体或其快捷方式即可快速完成所有DAX语句的格式化。

*注意：该模式仅针对于PBIRS，如果你使用标准版PBID，你依然需要在External Tools选项卡处运行它。*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118101731544.png#pic_left)

PBIRS版本要求：Power BI Report Server (2020年10月）或以上版本。

***End~***



