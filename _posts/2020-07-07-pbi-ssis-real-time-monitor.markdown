---
layout: post
title:  PBIRS 实现SSIS作业实时监控
date:   2020-07-07 06:03:50 +0000
image:  01.jpg
tags:   [Report Server,SSIS,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---


*本文将会分享一个我在过去做的一个ETL作业实时监控的报表项目，它利用Power BI Report Server (PBIRS) 直连 SQL Server 中的 SSIS数据库，实现对SSIS作业流的监控与管理。下文分享了项目的大体流程和现成代码。*

### 效果预览

此处提供一个脱敏截图，仅供参考。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200707144805587.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

你可以在此报表查询到最近所有的SSIS包任务的执行情况（你可以在SSMS修改SSIS数据库的历史数据记录范围），包括SSIS项目根目录，连接字符串，执行时间等等，你还可以像上图一样筛选出当日所有报错的任务，在左侧选择对应的包任务，右侧就可以展示该包的执行过程，这方便ETL开发者迅速找到报错的步骤以及原因，同时也便于开发者进行项目的调优。

### 实现方式

实现的过程并不复杂，关键是需要弄清楚SSIS数据库的字段逻辑。经过我此前的整理，在此整理出四段SQL查询，它们分别对应Power BI内不同的表。

**1. Executions**

主表，记录了所有包任务的执行情况。

```SQL
SELECT  A0.[execution_id]
      ,[folder_name]
      ,[project_name]
      ,[package_name]
	  ,CONVERT(VARCHAR(12),A1.[start_time],114) AS [Start Time]
	  ,CONVERT(VARCHAR(12),A1.[end_time],114) as [End Time]
	  ,DATEDIFF(ss,A1.[start_time],A1.[end_time]) as ExeTime
      ,CAST(A1.[start_time] AS date) as [Start Date]
	  ,[Execution Result] = CASE A3.[execution_result]
        WHEN 0 THEN 'Success'
        WHEN 1 THEN 'Failure'
        WHEN 2 THEN 'Completion'
        WHEN 3 THEN 'Cancelled'
        END
        ,[execution_result] as [execution_result_type]
  FROM [SSISDB].[internal].[executions] A0 WITH(NOLOCK)
  LEFT JOIN [SSISDB].[internal].[operations] A1 WITH(NOLOCK)
  ON A0.[execution_id] = a1.[operation_id]
  LEFT JOIN [SSISDB].[internal].[executable_statistics] A3 WITH(NOLOCK)
  ON A0.execution_id = A3.execution_id
```

**2. Executions_Parameter**

该表包含所有任务的连接字符串，属于敏感数据，建议实施RLS。

```SQL
    SELECT [execution_id]
                 ,[parameter_name] AS [Parameter Name]
                 ,[parameter_value] AS [Parameter Value]
      FROM [SSISDB].[internal].[execution_parameter_values]  WITH(NOLOCK)
      where [sensitive] = 0
```

**3. Executable**
   
该表记录了所有包的路径，以及执行日志

```SQL
SELECT  [execution_id]
      ,A0.[executable_id]
      ,[execution_path]
	  ,[project_id]
      ,[package_name]
      ,[package_path_full]
      ,[executable_name] as [Executable Name]
      ,[package_path]
      ,[execution_result] as [Execution Result Code]
      ,[Execution Result] = CASE A0.[execution_result]
        WHEN 0 THEN 'Success'
        WHEN 1 THEN 'Failure'
        WHEN 2 THEN 'Completion'
        WHEN 3 THEN 'Cancelled'
        END
      ,CONVERT(VARCHAR(12),A0.[start_time],114) AS [Start Time]
      ,CONVERT(VARCHAR(12),A0.[end_time],114) AS [end Time]
  FROM [SSISDB].[internal].[executable_statistics] A0 WITH(NOLOCK)
  LEFT JOIN [SSISDB].[internal].[executables] A1 WITH(NOLOCK)
  ON A0.[executable_id] = A1.[executable_id]
```

**4. Operations**

最后一张表，也是最大的一张表，记录了所有包任务的具体执行步骤以及对应信息。

```SQL
SELECT  [operation_message_id]
      ,A1.[operation_id] AS [execution_id]
	  ,[object_id] AS [project_id]
      ,[message_time]
      ,[message_type]
      ,[message]
  FROM [SSISDB].[internal].[operation_messages] A0 WITH(NOLOCK)
  LEFT JOIN [SSISDB].[internal].[operations] A1 WITH(NOLOCK)
  ON A0.[operation_id] = A1.[operation_id]
  where [object_id] is not null
```

在Power BI Desktop  获取这些表后，就可以进行建模，你可以参考以下架构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200707151446979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

### 在最后

接下来只需要完成可视化的部分即可。此外，该方案不仅适用于针对于本地部署的PBIRS，使用云端版Power BI Desktop, 只需要配置好本地网关，即可发布到Power BI Service使用。
