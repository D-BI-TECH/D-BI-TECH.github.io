---
layout: post
title:  Power BI Embedded 开发提要
date:   2021-02-01 01:03:50 +0000
image:  26.jpg
tags:   [Power BI,Azure,ASP.NET]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---


本文主要讲解Power BI Embedded开发过程中的一些要点，包括文档中未明确提及的步骤，一些可能会遇到的BUG的解决方案，以及个别功能具体的实现方法。

请注意，这是一篇**补充性**文章，既然是“提要”，自然不会是Step-by-Step的教程，我建议读者已对Power BI Embedded有了初步的了解。**跟随[官方文档](https://docs.microsoft.com/en-us/power-bi/developer/embedded/embed-sample-for-customers?tabs=net-framework)**，并结合本文，相信能够**帮助大家在开发过程中更加顺利**，少走些弯路。文末还附上了相关教程链接，里面包含了更详尽的资料。

此外，以下所有内容均为“Embed for customers (App-owns-data)”, 如果是面向组织的嵌入式开发可参考[此文档](https://docs.microsoft.com/en-us/power-bi/developer/embedded/embed-sample-for-your-organization)，然实际开发过程差异不大，本文亦可参考。

### 示例

首先，我**强烈建议读者到[此网站](https://microsoft.github.io/PowerBI-JavaScript/demo/v2-demo/index.html#)体验下官方做的示例**，也可以直接谷歌“Power BI Embedded Playground”，里面提供了许多很酷的示例，还提供了教学文档和视频：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021020117240077.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

下图是本人开发的示例，包含如下功能：

- 报表选择
- 数据筛选
- 切换主题
- RLS（基于角色的行级别安全性）
- 切换到编辑模式 编辑&保存 报表
- 显示&关闭 书签
- 进入&退出 全屏模式
- 页面刷新
- 查询最近一次数据刷新时间
- 执行数据刷新
- 切换到仪表板
- 仪表板显示模式
- 更多（..)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201172544253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

以上示例对于Power BI一般用户而言，功能已足够强大，但实际上能够实现的功能也远不止这些，此示例还仅仅是雏形，这意味着基于同样的技术实现原理，我们都可以实现许多完全超乎想象的功能，比如官方示例中弹出的具备发送邮件功能的自定义子菜单，以及自定义可视化分析等等。

*该示例暂时部署于个人的本地服务器，因此你还无法访问它，后期会录制视频来展示各功能效果*

现在，请跟随[文档](https://docs.microsoft.com/zh-cn/power-bi/developer/embedded/embed-sample-for-customers?tabs=net-framework)完成1到8步骤，并阅读以下提要。

### 前期工作提要

1.Power BI Embedded不是简单的iframe嵌入。你虽然可以简单地在PBI Service将报表发布到Web，然后在HTML页面里嵌入iframe，但这并非真正意义上的Power BI Embedded，因为你将不能控制权限，以及利用官方提供的强大的PBI JS库来实现丰富的交互功能等等。要实现这些效果，**必须首先获取有效的嵌入令牌**，这是前期工作的重点。

2.测试阶段，**建议选择Master Account模式**（PPU，Pro或Pro试用账户）。 Power BI Embedded拥有两种验证模式，Service Principal以及Master Account,  按官方说法，后者不能用于生产环境，且只能生成有限数量的嵌入令牌，但由于你可以使用该账户登录Power BI Service, 因此这能够方便开发者进行相关调试。Service Principal可以以用户身份加入安全组，以及**在PBI工作区中指派权限（这是必要步骤，如图）**，但其本身不能登录到Service。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201222600251.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

应用程序上线后，建议将验证方式改为Service Principal，使用该方式不需要在配置文件中提高账户和密码值，但需要提供Client Secret，这可以在Azure上创建并指定有效期：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201221718538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

此外，在PBI Service租户设置处启用“Allow service principals to use Power BI APIs”后（文档第6步），推荐将其**应用到安全组**中（安全组可由管理员在Azure创建）:

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021020122201310.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)



### 开发部署答疑


1.获取嵌入令牌失败

报错信息：401 Unanthorized: Error while retrieving Embed URL

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201224258915.png)

解决方案：在Azure Portal为App添加Power BI Service权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201224326416.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

*注：如果返回403 Forbidden，则需在PBI工作区追加App的权限（上文已有提及）*

2.‘$' 未定义

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201224900508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

解决方法：不要使用IE浏览器，改用较新版本的Chrome，火狐或Edge浏览器，并在HTML中引入Jquery：

```SQL
<script src="/scripts/jquery.js"></script>
```

3.Server Error in '/' Application

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201225759529.png)

解决方法：使用Package Manager安装.net包: 

```SQL
Install-Package Microsoft.Net.Compilers -Version 3.7.0-2.final
```


若问题依然存在，则需在IIS将其转为应用：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021020123002892.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

4.powerbi is not defined

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201230309723.png)

解决方法：将安装好的Powerbi.javascript包中的script文件夹复制到项目根目录

5.报表转换为编辑模式报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201235649586.png)

解决方法：

JS

```SQL
//在Config追加
permissions: models.Permissions.All
```

ASP.NET

```SQL
var generateTokenRequestParameters = new GenerateTokenRequest(TokenAccessLevel.View);
```

改为：

```SQL
var generateTokenRequestParameters = new GenerateTokenRequest(TokenAccessLevel.Edit);
```

*此处非常期待读者反馈您所遇到的问题，以及解决方案 (如果有) ，我会追加相关问题，以丰富这部分内容，帮助更多的人。*

### 【附1】特定功能实现方式

Power BI Emdedded Playground的原代码有些较为复杂，不易懂，此处仅举例一二，一通百通

1.实现全屏

HTML

```SQL
<button id="r_fullscreen" type="button" onclick="fullscreen()">全屏</button>
```

JS

```SQL
 function fullscreen() {
     var embedContainer = document.getElementById('embedDiv');
     report = powerbi.get(embedContainer);
     report.fullscreen();
     return;
 };
```

2.鼠标悬浮获取数据集最近一次更新时间

HTML

```SQL
<button id="d_refresh" title="
            上次刷新开始时间: 
            <% = this.refresh_startTime %>
            上次刷新完成时间: 
            <% = this.refresh_endTime %>" 
type="button" onclick="refresh()">刷新数据</button>
```

ASP.NET

```SQL
public string refresh_startTime;
public string refresh_endTime;
using (var client = new PowerBIClient(new Uri(Configurations.ApiUrl), Authentication.GetTokenCredentials()))
{
	var report = client.Reports.GetReportInGroup(Configurations.WorkspaceId, new Guid(ddlReport.SelectedValue));
	var refresh_history = client.Datasets.GetRefreshHistoryInGroup(Configurations.WorkspaceId,report.DatasetId);
	refresh_startTime = refresh_history.Value[0].StartTime.Value.ToString();
	refresh_endTime = refresh_history.Value[0].EndTime.Value.ToString();
}
```


### 【附2】相关资料

[Power BI Javascript 文档](https://github.com/Microsoft/powerbi-javascript/wiki)

[Power BI Developer in a Day course](https://docs.microsoft.com/en-us/power-bi/learning-catalog/developer-online-course)

[Power BI Developer in a Day 视频合集](https://www.youtube.com/playlist?list=PL1N57mwBHtN1AGWHnJMhtvJCIG_IlC07D)

[Power BI Embedded 实施RLS](https://docs.microsoft.com/en-us/power-bi/developer/embedded/embedded-row-level-security)

[RADACAD Power BI Embedded文章系列](https://radacad.com/tag/power-bi-embedded)

[Power BI Embedded 定价 (Azure版)](https://azure.microsoft.com/en-us/pricing/details/power-bi-embedded/)

-----------------

**关注我： [知乎](https://www.zhihu.com/people/zhang-zhe-hong-01/posts)  [Power BI官方社区](https://community.powerbi.com/t5/user/viewprofilepage/user-id/220984)**
