---
layout: post
title:  Power BI REST API实战教程：PowerQuery为例
date:   2020-10-02 01:03:50 +0000
image:  14.jpg
tags:   [REST API,Power BI,Power Query]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

*本文是D-BI之Power BI REST API系列第二篇，讲解用经典的方式，即[文档](https://docs.microsoft.com/en-us/power-bi/developer/embedded/register-app)中介绍的方式来注册一个AzureAD应用，并通过此应用来访问和使用Power BI REST API，最终实现利用PowerQuery获取Power BI Service的所有数据集*

### 前述

通过上文《[Power BI REST API有多强大？PBI开发者必读](https://d-bi.gitee.io/pbi-rest-api-introduction/)》我们得知PBI API带给我们的强大能力，但国内尚无任何使用PBI API的专门教程，尽管国外有较丰富的教程资料，比如David发布的《[Configuring Postman for Use with the Power BI REST API](https://dataveld.com/2020/05/09/using-postman-with-the-power-bi-rest-api/)》，但其中个别步骤已不能适用，而且也没有必要的注释来使大家避坑，因此本文以中文教程和避坑为亮点带领大家使用PBI API。需要注意的是，该技术的重点在于如何获取有效的访问令牌，只要能够做到这点，后期无论是使用Postman, Python, PowerShell还是PowerQuery来调用API，都是大同小异的。首先让我们注册一个PBI应用程序（亦可参照[文档](https://docs.microsoft.com/en-us/power-bi/developer/embedded/register-app)）。

*注：本文要求您已拥有Azure账户*

### 注册Power BI应用程序

首先我们使用[Power BI应用注册工具](https://dev.powerbi.com/apps)，跟随下图向导完成应用创建：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930115032408.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

填写必要信息，此处选择Native即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020093011564429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

权限处可直接全选，点击注册后会生成应用ID：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930115821197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

将此应用ID保留即完成创建。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020093012003621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

这一步暂时没有需要特别说明的地方。



### 获取API授权代码

到这里，国外某教程会让你在浏览器输入类似如下链接来完成用户授权：

```SQL
https://login.microsoftonline.com/common/oauth2/authorize?client_id=<<CLIENT_ID>>&response_type=code&redirect_uri=http://localhost/redirect/&response_mode=query&scope=openid&state=12345
```

没错，我们需要做这样做，但还不是时候。这里必须先到Azure Portal完成Redirect URL的设置，否则你将会在授权完成后得到如下报错：

```SQL
AADSTS50011: The reply URL specified in the request does not match the reply URLs configured for the application <<CLIENT_ID>>
```

打开Azure Portal，到AAD界面，找到您在上一步创建的应用，按下图操作，增加Redirect URL。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930160601905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

然后，顺便到API Permissions选项卡，我们还需要增加一些必要的权限，这里也是国外某教程忽略的地方。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930161025895.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

增加下图所示权限，然后点击【Grant admin consent for <租户名称>】

*Grant admin选项为灰色的说明您账号不具备足够权限，需要您组织Azure全域管理员或其他有权限的用户代为操作*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930161318136.png#pic_center)

到Certificates & secrets选项卡，增加Client Secret，这方便我们后续使用client_credentials的方式通过凭据验证，而非使用password的方式在请求中将您Azure的账户和密码指定，因为从安全角度考虑这样做更保险：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930163941584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)


完成以上步骤，我们才能打开浏览器，输入以下链接完成用户授权：

```SQL
https://login.microsoftonline.com/common/oauth2/authorize?client_id=<<你的CLIENT_ID>>&response_type=code&redirect_uri=<<你的Redirect URL>>&response_mode=query&scope=openid&state=12345
```

出现以下界面，直接点击【Accept】。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930162052143.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

由于我们的本地主机上没有运行任何web应用程序，我们将收到如下错误页面，这是正常现象。复制并保存右下角Requested URL处在单词code之后的文本，此即API授权代码，可留待后续使用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930162528884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)


### 使用PowerQuery获取访问令牌

现在，我们可以打开Power BI Desktop的PQ编辑器，构建我们的请求来获取访问令牌，一旦我们获取到了访问令牌，就可以享受到PBI API带给我们的强大能力。如下，在PowerQuery编辑器输入以下M语句，并在对应的参数上输入您应用的信息, 此处命名为GAT()：

```SQL
() =>
let
  apiUrl = "https://login.windows.net/<<输入您的租户ID>>/oauth2/token",
    body = [
          client_id=<<输入应用ID>>,
          grant_type="client_credentials",
          code=<<在此处输入上一步骤保存的API授权代码>>
 ,
          client_secret=<<输入client_secret>>,
          resource=<<输入Application ID URI，在Overview选项卡可找到>>
],

  Source = Json.Document(Web.Contents(apiUrl, [Headers = [Accept = "application/json"],
 Content = Text.ToBinary(Uri.BuildQueryString(body))])),
 access_token = Source[access_token]
in
access_token
```

点击【调用】后，我们就能获取到一长串的文本，即访问令牌：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930165308133.png#pic_center)

### 使用PowerQuery调用PBI API
成功获取访问令牌，代表我们已经成功了80%。现在我们只需要使用[PowerBI API文档](https://docs.microsoft.com/en-us/rest/api/power-bi/)中提供的接口即可实现对应的效果。现在我们以[获取Power BI Service上的所有数据集](https://docs.microsoft.com/en-us/rest/api/power-bi/datasets/getdatasets)为例进行演示。在PQ编辑器输入如下M语句，在请求头调用GAT()函数获取访问令牌。

```SQL
let
    Source = Json.Document(Web.Contents("https://api.powerbi.com/v1.0/myorg/datasets", [Headers=[Authorization="Bearer "&GAT()]])),
    value = Source[value],
    #"Converted to Table" = Table.FromList(value, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"展开的“Column1”" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"id", "name", "addRowsAPIEnabled", "configuredBy", "isRefreshable", "isEffectiveIdentityRequired", "isEffectiveIdentityRolesRequired", "isOnPremGatewayRequired", "targetStorageMode", "createReportEmbedURL", "qnaEmbedURL"}, {"Column1.id", "Column1.name", "Column1.addRowsAPIEnabled", "Column1.configuredBy", "Column1.isRefreshable", "Column1.isEffectiveIdentityRequired", "Column1.isEffectiveIdentityRolesRequired", "Column1.isOnPremGatewayRequired", "Column1.targetStorageMode", "Column1.createReportEmbedURL", "Column1.qnaEmbedURL"})
in
    #"展开的“Column1”"
```

运行后如下所示，成功获取PBI账户工作区内所有数据集的信息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200930170902718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

你还可以使用其他查询语句来获取其他信息，比如要获取某数据集的刷新历史记录，只需将Web.Contents()内第一个参数改为以下URL即可，具体[参考API文档](https://docs.microsoft.com/en-us/rest/api/power-bi/datasets/getrefreshhistory)：

```SQL
https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes
```


### 总结与后续

使用PBI API，除了读取信息还可以执行一些任务，比如刷新数据集等操作，我已在[上文](https://d-bi.gitee.io/pbi-rest-api-introduction/)总结了它的主要功能，但这些操作建议使用PowerShell或Python来完成，如果是用于测试也可也使用Postman，还是开头那句话，此处使用什么工具调用API区别不大，请求所需的参数和发送方式没有什么区别，关键在于成功获取访问令牌。

Power BI REST API的实战教程到此结束，也许部分读者会觉得整个过程较为繁琐，事实上，我还有另一套方案，使用Python只需三十几行代码即可完成PBI API的调用，不需要创建PBI应用也不需要到Azure做各种麻烦的设置，具体如何实现，我会在下文（D-BI之Power BI REST API系列的第三篇）具体说明。