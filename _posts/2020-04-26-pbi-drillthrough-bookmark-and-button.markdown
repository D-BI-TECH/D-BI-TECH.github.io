---
layout: post
title:  Power BI 关于钻取，书签与按键
date:   2020-04-26 08:03:50 +0000
image:  05.jpg
tags:   [Power BI,可视化]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

经历了忙碌的四月，到月底，终于能抽出一丢丢时间写篇短博。本文的主题是Power BI中的钻取，书签以及按键，这些都是微软很早就已经发布的功能，但这三者结合应用，可以为PowerBI报表实现更强的交互性。

书签和钻取的结合为开发者在PowerBI报表实现弹窗式设计提供了可能，书签可以保存页面的一种状态，比如不同可视化之间的上下层关系，以及隐藏和显示的状态，而按键则可以被赋予导航到书签的属性（近期也为按键提供了页面钻取的功能，将在下文讲述），由此可以使得用户能够直接利用按键的方式来决定哪些可视化可以显示，哪些隐藏，按键本身也是可视化，也可以被其他按键触发的书签效果所隐藏，由此使得PowerBI报表的设计比以往有更多的可能性和想象空间。就比如我在下图展示的那样：

![点击前](https://img-blog.csdnimg.cn/20200426205644986.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

点击“K线图模式”后，就可以切换到下图的布局：

*（如果你是D-BI的死忠读者，说明你的理解力和基础水平都相对高些，这一步的操作，即使你此前不了解也一定能通过上文的叙述摸索出来，此处就省去具体步骤了，这不是本文的主题，在这里我们崇尚浓缩，摒弃冗余）*

![点击后](https://img-blog.csdnimg.cn/2020042620563689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

*注：细心的你可能会留意到上图Y轴实现了不同颜色的轴标签以及自定义的辅助线，这并不是用了第三方的高级可视化控件，而是用的R Visual，用R代码构建的，且此图还可以利用DAX构建的度量值来控制其显示的效果（颜色，线条等等），关于此，我会在日后的文章中进行说明。*

因此，通过结合书签与按键，用户可以直接改变报表的布局以及显示的图表类型，而不是像以往的那样，只能由开发者定义好图表类型。
顺便提一句，就在我写此文之时，Danill在Linkedin分享了Power BI最新版的一个重要特性，PowerBI自带的可视化控件，即使无需利用按键也能部分实现图表类型的切换了，这的确是一个令人感到惊喜的功能：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426212422306.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

回到正题，当我们把书签，按键结合后，再加上一个钻取，就能开发出应对更复杂情景下的企业报表。比如我们可以做一张总表，里面列出所有店铺的销售或财务数据，我们可以利用钻取特性到子页面查看每家店铺的明细数据，而总表和子报表都可以设定多个书签，利用按键切换图表和布局，这可能很完美，以本文案例的金融股市数据为例，对应的报表结构设计如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426213250930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

大盘数据对应总表，可以切换不同的布局，子报表也一样，可以切换布局和图表，这看起来很完美，但当你真正实施时你可能会遇到一个问题，比如用户在下图界面钻取到子报表--美的集团，我们会到达美的集团的股票分析界面，当用户在子报表（即上图个股分析）点击切换图表的按键后（由K线图切换到线图），数据却显示了第一支股票--平安银行的数据，而非美的集团。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426213540517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

原因何在？这是因为当你在设置书签时，书签的状态就是平安银行（假如），书签不会理会你的钻取筛选条件，因为书签默认保存了当时的数据条件，为了解决此问题，你必须在设置书签处，将“Data”的勾选取消，这样当你在总表钻取到店铺B的数据时，在子报表切换书签就还能继续显示店铺B的数据，而不是切换回店铺A：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426214831263.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

钻取和按键的结合在新版的Power BI里得到实现，这里通常的用法是，用户可以点击在表格或矩阵或是其他受支持图表中的数据点，比如在前文中选择“美的集团”，这样就可以激活按键，用户就可以通过点击该按键下钻到子报表，而非原来的右键钻取方式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426215141274.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

透过表面的功能思考，你会发现这同时也给予我们一种新的PBI报表设计思路---流程式报表设计。用户在使用报表时，不会面对一大堆报表页面而手足无措，当用户打开报表时，只会看到一个页面—首页，然后通过其不同的选择或钻取到达其他页面，报表开发者可以根据其需求设计更合适的流程加强用户体验，分析师也可以借助报表的流程式设计向用户传递其数据中的故事，你甚至可以利用此功能在PowerBI做出调查问卷的效果,以及“阅读条款”式的首页界面：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426220229848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

点击"已阅读上述内容"的部分是通过表格控件实现的，因为隐藏了周围的虚线，所以完全看不出是表格。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426220239128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

以上的这些所有特性，包括微软Power BI开发团队高频率的更新，都在向我们传递出一种用户主导的思路，报表的开发者利用了钻取，书签以及按键三大元素的结合，完全可以开发出自由度更高，交互体验更强的BI报表。


