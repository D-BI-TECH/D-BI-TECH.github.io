---
layout: post
title:  PBI Report Server建立报表警报机制
date:   2019-12-02 06:03:50 +0000
image:  11.jpg
tags:   [PBI Report Server,SQL Server]
---

在Power BI Services, 只要你拥有Pro License，或者数据仪表板位于Premium空间，你就可以为特定的数据磁贴设置警报，当磁贴中的数据满足你所设定的条件时，Power BI Services就会向你推送通知，更进一步地，你还可以使用Microsoft Flow (现已改名为Power Automate) 将通知发送到你或你的同事的邮箱中，关于此，你可以参考[此文]({{site.baseurl}}/microsoft-flow-for-pbi/)进行设置。但对于那些没有购买License，使用Power BI报表服务器的用户而言，如何实现这个功能？

事实上，使用Power BI报表服务器是可以实现这个功能的，前提是你的报表数据源也在关系型数据库中，其原理是在SSMS建立一个Job，判断Power BI的数据源中是否存在满足条件的行，如果满足的话，就执行警报邮件作业，否则就不执行。但这里存在的一个问题是，当Power BI报表数据刷新失败时，就可能会造成判断结果与Power BI报表的实际情况不一致，解决的办法同样是设定一个监控，就是监控报表服务器数据库日志，流程是：执行数据警报作业，如果查询到满足条件的数据则发送数据警报邮件，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202234812531.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

具体的实现方法并不在报表服务器设置，而是首先打开SSMS。打开后，连接到报表服务器数据库所在的数据库实例，然后在Object Explorer找到"Management", 点击"Configure Datebase Mail":

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202234823911.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

选择管理邮件账户：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202234842882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

然后新建账户：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202234850714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

按下方图示填写以下信息后，点击下一步完成创建向导：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202234900163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

右键SQL Server代理，选择属性，勾选"Enable mail profile",然后在"Mail Profile"处选择刚刚新建的邮件账号：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120223490976.png)

最关键的步骤，就是利用我们的账户，使用SQL执行邮件任务了，在下方代码基础上替换上你的代码（此处只是最基本的示例，你需要根据你自己的需求进行调整，比如声明变量等等)：  

>```SQL
IF --"在此设定条件，比如当某个数值大于100时触发下面的代码，否则不执行"
BEGIN   
    EXEC msdb.dbo.sp_send_dbmail 
        @profile_name = '<这里填写你所建立的邮件账户名>', 
        @recipients = '<这里填写收件人邮箱>' , 
        @body = '<这里填写邮件内容>', 
        @subject = '<这里填写邮件标题>' ,
		@query = N'<如果你要邮件显示报表服务器数据库中的数据，
                             在此填写你要执行的SQL查询，可选>',
		@attach_query_result_as_file = 0, --"0"意为不设附件,"1"为设置附件，
                                                      -- 默认txt格式，可选
		@query_result_header = 1, --"0"意为结果不要表头，"1"为包含表头，可选
		@query_result_width = 120; --结果的列宽，可选
		END
>```

最后，把SQL复制粘贴到你所设置的job的"Step"中，设定好Schedule，定时执行该任务。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202234917775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

到此，SQL Server就会定时检测特定数据是否满足相应的条件，然后决定是否发送通知邮件。此处也许有人还不太明白，我就拿监控报表刷新为例，因为报表服务器数据库大家都一样，方便各位自行测试。对应SQL如下：

>```SQL
  IF EXISTS(
  SELECT TOP 1
        [Name]
        ,[LastStatus]
        ,[LastRunTime]
        FROM [<报表服务器数据库名称>].[dbo].[Subscriptions] A0 WITH(NOLOCK)
        LEFT JOIN [<报表服务器数据库名称>].[dbo].[Catalog] A1 WITH(NOLOCK)
        ON A0.Report_OID = A1.ItemID
        LEFT JOIN [<报表服务器数据库名称>].[dbo].[Users] A2 WITH(NOLOCK)
        ON A0.OwnerID = A2.UserID
        where [EventType] = 'DataModelRefresh'
        and [LastStatus] <> 'Completed Data Refresh'
		)
	BEGIN   
    EXEC msdb.dbo.sp_send_dbmail 
        @profile_name = 'Davis', 
        @recipients = '<收件人地址>' , 
        @body = 'Power BI Report Refresh Alert', 
        @subject = 'PBI Alert' ,
		@query = N'SELECT 
        [Name] as [ReportName]
		,[Username] as [Publisher]
        ,[LastStatus]
        ,[LastRunTime]
        FROM [<报表服务器数据库名称>].[dbo].[Subscriptions] A0
        LEFT JOIN [<报表服务器数据库名称>].[dbo].[Catalog] A1 
        ON A0.Report_OID = A1.ItemID
        LEFT JOIN [<报表服务器数据库名称>].[dbo].[Users] A2
        ON A0.OwnerID = A2.UserID
        where [EventType] = ''DataModelRefresh''
        and [LastStatus] <> ''Completed Data Refresh''',
		@attach_query_result_as_file = 0,
		@query_result_header = 1,
		@query_result_width = 120;
		END
>```

把该SQL放入Job中执行，这样每当数据刷新失败时，报表管理员就能及时收到通知邮件，及时去排查错误原因了。下图是收到的示例邮件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202235006102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

现在我们已经成功建立了数据刷新警报机制，但邮件的排版却不让人满意，解决方法是把查询出的数据结果以HTML形式展示，需要使用SQL声明变量将日志查询结果拼成html格式，结果传入body变量中，再在上述代码增加一行"@body_format ='HTML'"，就可以使邮件以html形式更加规范地显示结果了，如下所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202235015656.png)

**喜欢本文请将本站分享给你的朋友或同事，或点击本站右下角"知",关注我的知乎账号，感谢支持**