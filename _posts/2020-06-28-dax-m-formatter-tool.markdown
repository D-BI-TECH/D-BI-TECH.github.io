---
layout: post
title:  一分钟格式化所有DAX及M语句
date:   2020-06-28 03:03:50 +0000
image:  12.jpg
tags:   [Power BI,DAX,Power Query]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

### 关于“DAX & M批量格式化工具”

作为Power BI报表开发者，DAX代码的可读性十分重要，几年前SQLBI推出了一个免费web服务：[daxformatter](https://www.daxformatter.com/)，它允许你粘贴你的DAX代码到输入框，一键助你完成代码的换行与缩进，提升代码可读性，最近，PQ里的M语言也有了类似的服务---[Power Query Formatter](https://powerqueryformatter.com/), 利用这些工具，可以帮你无需手工操作就能完成换行缩进，提升你的报表开发效率。不过，这些工具的美中不足是不能一次性格式化多个公式，即使可以，一个个粘贴公式也是不方便的，因此，我开发了一个小工具---"DAX & M批量格式化工具"，它可以帮你一次性格式化你的PBIX文件内的所有DAX公式，包括所有表的度量值以及计算列。工具截图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020062810232511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

(注: 同时也可以帮你格式化所有表的M语句, 这里需要说明的是，该格式化工具的原理是通过向daxformatter以及powerqueryformatter网站发送POST请求并返回格式化的结果写入PBIX中的配置文件以完成全部代码格式化，但由于powerqueryformatter网站处于测试状态，接口还存在问题，因此格式化M语句功能暂不可用）

该工具对于那些大型Power BI报表项目的作用十分明显，因为你可能会有上百个DAX度量值以及计算列等等，一个个的格式化将是十分耗时且效率低下的，而通过该工具，仅需1分钟即可完成报表文件中全部DAX的格式化，使你把更多精力专注于更重要的环节：模型设计和效率优化。

### 如何使用

第一次使用，需要多一个小步骤，打开Power BI Desktop, 打开选项：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628103756272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

基于作者本人的Desktop版本（2020年5月），此时需要在预览功能处勾选如下功能，然后重启Power BI Desktop：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628104006139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

接下来是工具的使用方法，以如下文件为例：

这是其中的Sales表的其中一个计算列：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628104801248.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

以及部分度量值：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628104847461.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628104941176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

还有很多其他计算列和度量值存在不同的表，此处不一一列举。现在关闭PBIX文件（推荐另存为PBIT格式，此处另存为PBIT命名为formatterTemp），修改文件的后缀为ZIP，解压文件，你会看到一个名为DataModelSchema的文件，我们的所有DAX以及M代码等等都在此文件中定义，因此工具的任务是通过程序修改该文件以达到一次性格式化目的。现在，右键复制该文件的路径：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628110648577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

打开“DAX & M批量格式化工具”后将路径粘贴到输入框，然后点击“执行”，等待几秒钟后会提示“已完成”：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628110440269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

现在文件夹里的DataModelSchema已是修改好的文件了，最后重新打开zip文件，将更新后的DataModelSchema拖入覆盖即可：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628111233292.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

最后，只需将文件ZIP后缀改回PBIT或PBIX，打开文件即可发现所有的DAX代码均已成功格式化！如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628111744158.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628111647491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200628111713163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 获取该工具

由于该工具的M格式化部分尚有未完成部分，因此暂时不会发布，感兴趣的读者可以关注我的[知乎账号](https://www.zhihu.com/people/zhang-zhe-hong-01)，点赞+收藏此文，并在评论区留下邮箱，后期完成会第一时间将工具安装包发送到您邮箱（注：该工具属于公益性质，不会收费，但也不会保证任何时效性，并且日后是否继续开发以及优化改进，增添新功能等等，还取决于社区及读者对该工具的需求程度及反馈数量）
