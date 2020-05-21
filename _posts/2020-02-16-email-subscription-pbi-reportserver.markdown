---
layout: post
title:  PBI Report Server邮件订阅进阶
date:   2020-02-16 01:03:50 +0000
image:  12.jpg
tags:   [Report Server,SQL Server,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

*【前述：本文主要讲解如何在Power BI 报表服务器中，为Power BI报表设置邮件订阅服务】*

如果你使用过Power BI Report Server 或 SSRS，我想你也许了解在报表服务器中可以为[分页报表](https://docs.microsoft.com/en-us/sql/reporting-services/reports/reporting-services-reports-ssrs?view=sql-server-ver15)设置邮件订阅服务，以实现报表使用者可以定期通过E-MAIL收到报表附件或报表链接（如果你不了解具体如何配置邮件订阅服务，那么你可以参考[此文]({{site.baseurl}}/Deployment-configuration-pbi-report-server/)），然而遗憾的是，截至目前最新版本（2020年1月版本），在Power BI报表服务器的门户网站中，并未提供为Power BI报表设置邮件订阅服务的功能，如下图对比所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215150127730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

然而在云端部署的Power BI Services中，邮件订阅及分享PBI报表的功能却是早在三年前就已经推出的基本特性。相比Power BI而言，微软对本地部署的Power BI (RS)一直未有积极的跟进，但这不代表用户就没有需求，相反，很多公司希望Power BI报表能够和分页报表一样，可以在报表服务器实现邮件订阅，统一管理。

幸运的是，就像[在PBI Report Server中实现报表预警机制]({{site.baseurl}}/set-email-alert-pbi-reportserver/)一样，在本地部署的Power BI中，我们不仅可以实现PBI报表的邮件订阅，而且可以实现得更完美。

### 前提：  

为了实现该功能，需要确保满足两个条件：你的报表服务器为分页报表创建邮件订阅的功能能够正常运行，这代表你在报表服务器Configuration Manager上的配置是正确的。如果你是一名纯粹的Power BI报表开发者，不了解[关于分页报表的基础知识]({{site.baseurl}}/introduction-pbi-reportBuilder/)，你需要打开Configuration Manager检查数据库和电子邮件设置并确保已经开启了SQL Server代理服务。其次，你已经在SSMS中配置了数据库邮件（具体步骤参考[此文]({{site.baseurl}}/set-email-alert-pbi-reportserver/)）。

### 实现：

实现该功能并不难，其原理与设置报表警报机制类似，都是通过调用SQL Server系统数据库或报表服务器数据库的存储过程来实现，事实上，作为测试，你完全可以复制以下我所提供的代码, 在必要处替换为你的相关信息，创建一个发送邮件的存储过程：

```SQL
USE [ReportServer]
GO
create procedure pbi_send_email_test
	   @ReportName nvarchar(100)
	  ,@recipients nvarchar(100)
	  as
begin
Set NoCount On;

DECLARE

@Content_head varchar(max),

@content varchar(max),

@Content_tail varchar(max)

Set @Content_head = 
'<body lang=ZH-CN link="#0563C1" vlink="#954F72" style=''tab-interval:21.0pt;
text-justify-trim:punctuation''>
<div class=WordSection1 style=''layout-grid:15.6pt''>
<br>
Please follow 
<a href='

set @content = 
'<在此输入你的PowerBI门户URL+文件夹名称(如果有)>'+replace(@ReportName,' ','%20')+'?rs:embed=true>'
+'this link'

Set @Content_tail = 
'</a>
to access your Power BI report<br>
</div>
</body>'

select @content = @Content_head + @content + @Content_tail

IF EXISTS(SELECT @content)

BEGIN
	  
    EXEC msdb.dbo.sp_send_dbmail 
        @profile_name = '<在此输入你在SSMS中配置的邮件账户名>', 
        @recipients = @recipients , 
		@importance = 'NORMAL',
        @body = @content, 
        @subject = @ReportName ,
		@body_format ='HTML';
		END
END
```

该存储过程所实现的效果是，你只需指定两个必要参数：报表名称和收件人邮箱地址：

*（当然你也可以在存储过程中添加和使用其他必要的参数，比如抄送人）*

```SQL
USE [ReportServer]
GO

DECLARE	@return_value int

EXEC	@return_value = [dbo].[pbi_send_email_test]
		@ReportName = N'<PBI报表名称>',
		@recipients = N'<收件人邮箱>'

SELECT	'Return Value' = @return_value

GO
```

 执行存储过程后， 就可以把你所指定的报表的访问链接发送给你所指定的收件人，报表以本文第一幅图中的basket analysis 2为例，收到邮件后如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215162058549.png)

成功收到邮件后，最后一步就是为邮件设置订阅计划，此处在[SSIS](https://docs.microsoft.com/en-us/sql/integration-services/sql-server-integration-services?view=sql-server-ver15)以及SSMS中都可以实现，如在SSMS中创建一个作业：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215162752189.png)

将上文执行存储过程的代码粘贴到“step”中，最后根据你的需求设置好订阅的时间计划即可：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215163318427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

至此，Power BI报表的邮件订阅就设置完成了，如果你要为其他用户订阅其他的报表，只需要填写好对应的参数即可，你也可以填写多个收件人，如果你能熟练使用SQL，通过改进原存储过程还可以实现更加强大的效果，实现全局的，自定义化程度高，多人共享的企业报表订阅解决方案。

### TIPS

此外，在存储过程中你能发现我使用了HTML对邮件进行排版，上文所示仅是最基本最简单的格式，你完全可以利用HTML为你的订阅邮件进行美化。如下图在原基础上添加图片及加粗部分文字：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215171117262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

如果你没有学过HTML或者懒于书写冗长的HTML标签也是没有关系的，有一个技巧是，你可以在Word中为你的邮件排版，然后以HTML格式另存，在文本编辑器中直接复制代码即可。你完全可以根据组织或你个人的偏好定制更加规范、简洁、美观的报表订阅邮件。


