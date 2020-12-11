---
layout: post
title:  Power BI Report Server 负载优化：报表定期清理程序
date:   2020-12-11 01:03:50 +0000
image:  22.jpg
tags:   [Power BI,PBIRS,Python]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

### 需求

在企业环境，管理层，以及各个部门都会需要各种类型的报表，无论是业绩，生产，财务，还是KPI。这些报表除种类繁多之外，其本身也有它的生命周期。

环境在变，需求在变，报表也随之在变。

对于BI部门而言，除了需要不断更改迭代的报表以及新的项目之外，你也许没有留意到，这些在报表服务器上越积越多的报表，实际上只有不到一半的报表处于活跃状态。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210220457376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)


随着业务的发展，我们有越来越多的报表在“积灰”，在相当一段时间内，都没有任何BI团队以外的用户访问，这不仅占用了服务器的磁盘空间，还使得整个报表服务器变得臃肿而不易管理，更重要的是，这些报表的数据刷新计划占用了我们服务器一部分的系统资源。在默认情况下，80%内存占用率是一道临界点，如果超过这个数值，就会影响到报表服务器的后台查询进程以及用户查看新报表的速度。

于是，摆在我们面前的一道需求，就是可以建立一个自动化程序，定期自动检查所有报表的活跃状态。比如查出有哪些报表超过90日无人访问，那么我们可以对这些报表做进一步的处理。

*Power BI 报表服务器内存临界点可调节，详见[此文档](https://docs.microsoft.com/en-us/sql/reporting-services/report-server/configure-available-memory-for-report-server-applications?view=sql-server-2017)*

### 什么是报表定期清理计划（RRS)

报表定期清理计划（Reports Removal Schedule）是本人定义的，用于解决上述需求的一种策略，旨在无需人为干预的情况下，自动清理无用的报表。

对于一份报表（此处我们特别指Power BI报表，因为部分以附件形式分发的分页报表其使用率难以被评估），每当BI团队之外的用户，在相当一段时间内没有访问某报表，此时系统自动向所有用户发送一封提示邮件，附上报表链接，并告诉用户是否继续需要使用此报表，否则报表会在某某日期后下线。如果在下线日期前, 没有任何用户再次访问该报表，那么系统强制性下线报表。

![AFTER](https://img-blog.csdnimg.cn/20201211102627809.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

*如图，实施该计划后，可大幅压缩闲置报表的占比*

下线报表主要有两种方式：

1. 其一是将报表移动到仅供BI内部人员使用的测试文件夹，并关停该报表的所有数据刷新计划, 相当于“暂停维护”；

2. 第二种办法则简单粗暴，直接从报表服务器删除报表。下图提供了该计划的流程，仅供参考：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207235634406.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)

无论哪种方法，我们都可以建立一套程序实现流程全自动化。下文介绍其技术原理。

*注1：部分读者可能会有疑问，如果我的某些报表本来就是一年才看一次的，实施该计划会不会导致误删呢？实际上，**本文的重点在于思路和最基本的技术实现方式**，至于这些细节问题，我们可以有很多办法；比如我们完全可以针对不同的报表设置不同的时间阈值，此处只是为了便于读者理解，简化了流程*

*注2：对于方法二，建议您组织已实施报表备份计划*

### 实现原理

此处以方法二为例，介绍如何实现检测出所有超过10天无人访问的报表，并删除它们。对于发送提示邮件，可以结合[此文中介绍的技术](https://d-bi.gitee.io/email-subscription-pbi-reportserver/)一同实现。实施此计划主要在于两点：

1. 检测所有Power BI报表的被访问状况
2. 删除这些报表

对于第一点，我们只需要使用SSMS连接到Power BI报表服务器的数据库实例，执行以下查询，即可找出10天内无任何人（管理员除外的）访问的Power BI报表名单：

```SQL
    SELECT NAME
    ,[ReportID]
    FROM
    [Catalog] M1
    LEFT JOIN
    (
      SELECT [ReportID]
          ,MAX([TimeStart]) AS LastVisitTime
      FROM [ExecutionLogStorage] P0
      JOIN [Users] P1
      ON P0.UserName = P1.UserName
      WHERE P0.Format = 'PBIX' 
      AND P1.UserType <> 0
      GROUP BY 
      [ReportID]
    ) M0
      ON M0.ReportID = M1.ItemID
      WHERE M1.Hidden = 0
      AND M1.Type = 13
      AND (CASE WHEN LastVisitTime IS NULL
    	  THEN DATEDIFF(DAY,CreationDate,GETDATE())
    	  ELSE DATEDIFF(DAY,LastVisitTime,GETDATE())
    	  END) > 10
```

执行以上查询，你就能找出满足条件的报表名单。此外，你也可以留意到这段SQL的逻辑，对于新发布的报表，它本身不会有“上次访问时间”，那么我们会以其创建时间为时间起点（如上文流程图所示）

在找出这些报表后，如何删除他们呢？此时我们就需要调用PBIRS的REST API，向报表服务器发送DELETE命令。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201211000207473.png)

有关调用PBIRS REST API的方法，可[参考此文](https://d-bi.gitee.io/pbi-reportserver-rest-api-with-postman-and-python/)。

下文提供了以Python语言为例的示例代码以供参考，该代码基于本人在实际项目中的代码简化而来，且实测有效。此处仅是基本用法，实际情况可能会复杂得多，但我们可以此为基础结合实际需求自行发挥。

### 完整代码示例

##### Python

```SQL
import requests
import pymssql
from requests_ntlm2 import HttpNtlmAuth


conn = pymssql.connect(
        host=<YOUR HOSTNAME>,user=<YOUR DB ACCOUNT>
        ,password=<YOUR DB PASSWORD>,database=<YOUR DB NAME>
        ,port=<YOUR DB PORT>,charset='utf8')

cursor = conn.cursor(as_dict=True)

cursor.execute(
    """
    SELECT NAME
    ,[ReportID]
    FROM
    [Catalog] M1
    LEFT JOIN
    (
      SELECT [ReportID]
          ,MAX([TimeStart]) AS LastVisitTime
      FROM [ExecutionLogStorage] P0
      JOIN [Users] P1
      ON P0.UserName = P1.UserName
      WHERE P0.Format = 'PBIX' 
      AND P1.UserType <> 0
      GROUP BY 
      [ReportID]
    ) M0
      ON M0.ReportID = M1.ItemID
      WHERE M1.Hidden = 0
      AND M1.Type IN (2,13)
      AND (CASE WHEN LastVisitTime IS NULL
    	  THEN DATEDIFF(DAY,CreationDate,GETDATE())
    	  ELSE DATEDIFF(DAY,LastVisitTime,GETDATE())
    	  END) > 90
      """)
      
result = cursor.fetchall()

id_list = []
for i in range(0,len(result)):
    id=str(result[i]['ReportID'])
    id_list.append(id)

username = <YOUR PBIRS ACCOUNT>
password = <YOUR PBIRS PASSWORD>
baseurl = 'http://<YOUR SERVER IP or HOSTNAME>//Reports/api/v2.0/'
queryurl = 'PowerBIReports'

header = {
    'username':username,
    'password':password,
    'Workstation':<YOUR HOSTNAME>
}

auth=HttpNtlmAuth(username,password)
 
for i in id_list:
    querydelete = baseurl + "PowerBIReports("+i+")"
    requests.delete(querydelete,auth=auth)'''
```

