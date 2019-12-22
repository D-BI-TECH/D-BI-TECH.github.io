---
layout: post
title:  PBI Report Server触发式数据刷新
date:   2019-12-22 00:03:50 +0000
image:  06.jpg
tags:   [Report Server,SQL Server,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
---

***注：了解包括PowerBI的最新资讯，或有相关技术问题，请关注知乎[微软BI技术资讯圈](https://www.zhihu.com/club/1189912932908257280?ab_signature=CiRBS0Fqc2xNRUNSQkxCZklPdmJXSGxBY1ZYWmN0WEQ2RURRaz0SIGE5MTkyMTI2ODM3MTJlZmY5ZjgxNTkxNGRhZjYzZjIxGhAIARIGNi4yNC4xGgQxNzky)。有资讯一起共享，有问题发帖讨论！***

最近有人问我，直接在报表服务器数据库进行操作是否会影响到PowerBI报表门户的设置，答案当然是肯定的，除了动态权限控制，[设置数据警报]({{site.baseurl}}/set-email-alert-pbi-reportserver/)以及监控报表用户等等，利用报表服务器数据库还能完成很多你在PowerBI Services难以做到的事情。本文的主题就是一个经典用例——利用Report Server数据库的存储过程，设置触发式的报表数据刷新。

### 什么是触发式数据刷新？

顾名思义，就是用事件触发PBI报表的数据刷新。比如，数据源发生了变动，新增，改动或者删除了一条数据，那么就激活了触发条件，让PBI报表立即刷新数据，使PBI报表数据保持最新，典型的流程如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222124218784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Nzk0NzE0,size_16,color_FFFFFF,t_70)

### 为何要使用触发式数据刷新？

在Power BI Report Server, Power BI报表的数据刷新主要分两种：刷新计划和DirectQuery。DirectQuery即直连，实时连接的好处自不用我说，但缺点是当报表计算逻辑复杂时，就可能严重影响报表性能，更不用说还不支持某些DAX函数了。刷新计划也分两种，共享计划和报表刷新计划，报表刷新计划又分一次性刷新和定时刷新，但不管哪种方式，都无法保证让用户打开报表就能看到最新的数据，也许你会为你的报表设置每隔一分钟刷新一次，但如果你的报表后期数据量变大，不仅增大刷新失败的风险，对服务器也会造成一定负担。

因此，触发式数据刷新成为了最佳解决方案，它结合了DirectQuery和刷新计划二者的优点，只有当数据源数据发生了变动，才会执行数据刷新，这既可以保证用户在打开报表时看到最新数据，又能减轻服务器负担，还不会影响到报表性能。

### 如何实现？

首先，我在数据库创建一个表作为示例。如下所示，我创建了一个新表，插入了三条数据——战国时期的四大名将之白起，廉颇和王翦：

>```SQL
CREATE TABLE [TriggerRefreshTest]
(
[No_] int,
[Name] nvarchar(10),
[Country] nvarchar(20)
)
Insert Into [TriggerRefreshTest]
values(
1, 'BaiQi', 'The Kingdom of Qin'
),(
2, 'LianPo', 'The Kingdom of Zhao'
),(
3, 'WangJian', 'The Kingdom of Qin'
)
>```

将数据导入到Power BI Desktop并发布，如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222124228488.png)

现在我们要实现的效果是，我在数据库中插入一条新数据--战国四大名将之李牧，然后自动触发刷新事件，使报表数据保持最新。
在[SSMS](https://docs.microsoft.com/zh-cn/sql/ssms/sql-server-management-studio-ssms?view=sql-server-ver15)，新建触发事件，如下示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222124244888.png)

然后用SQL编写触发脚本，没学过SQL的读者也没关系，你可以直接使用以下代码，我在编写该脚本时做了精简和优化，你只需要在下面 [REPORT_NAME] 处换上你的PBI报表名称即可使用：

*备注：如果你的报表服务器数据库名称不同，换上你的数据库名称即可*

>```SQL
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
IF OBJECT_ID
(
    N'trigger_data_refresh',
    N'tr'
) is not null
DROP TRIGGER trigger_data_refresh;GO
CREATE TRIGGER trigger_data_refresh 
ON [TriggerRefreshTest]
AFTER INSERT
AS 
SET NOCOUNT ON;DECLARE @REPORT_NAME NVARCHAR
(
    50
),
@REPORT_ID VARCHAR
(
    100
),
@SUBSCRIPTION_ID VARCHAR
(
    100
)
SET @REPORT_NAME = 'WarringStates' --此处换上你的PBI报表名称
SET @REPORT_ID = 
(
    SELECT TOP 1 [ItemID]
    FROM [ReportServer].[dbo].[Catalog]
    WHERE [Name] = @REPORT_NAME
)
SET @SUBSCRIPTION_ID = 
(
    SELECT TOP 1 SubscriptionID
    FROM [ReportServer].[dbo].[ReportSchedule]
    WHERE [ReportID] = @REPORT_ID
)
BEGIN
WAITFOR DELAY '0:0:3'
exec [ReportServer].dbo.AddEvent 
@EventType='DataModelRefresh',
@EventData=@SUBSCRIPTION_ID
END
GO
>```

最后，运行该脚本，我们的触发就设置完成了，现在我们向数据源表插入一条新数据吧：

>```SQL
Insert Into [TriggerRefreshTest]
values(
4, 'LiMu', 'The Kingdom of Zhao'
)
>```

回到Power BI Report Server中的报表页面，如前述步骤无误，则此时数据已经刷新完成，重新打开报表或点击下图"Refresh"刷新缓存，如你所见，刚刚插入的新数据就立刻反映到报表中了！到此我们成功设置了触发式数据刷新，修改及删除数据原理相同在此省略：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191222124251938.png)

### 顾虑与启示

如果数据源表是多人维护的话，就可能出现多人同时修改源表的情况，此时触发刷新可能会造成问题（比如锁表），解决的办法是设置强制刷新最小间隔，比如下一次刷新必须至少在上一次刷新的一分钟之后。此外，利用报表服务器数据库，我们还能实现很多其他功能，有兴趣的读者可自行探索，如果有不懂的地方可在评论区留言或到知乎[BI技术圈](https://www.zhihu.com/club/1189912932908257280?ab_signature=CiRBS0Fqc2xNRUNSQkxCZklPdmJXSGxBY1ZYWmN0WEQ2RURRaz0SIGE5MTkyMTI2ODM3MTJlZmY5ZjgxNTkxNGRhZjYzZjIxGhAIARIGNi4yNC4xGgQxNzky)发帖🙂

***END~***
