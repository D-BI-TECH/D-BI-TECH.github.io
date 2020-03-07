---
layout: post
title:  PBI：用Python实现Dynamics 365数据集的增量刷新
date:   2020-02-25 02:03:50 +0000
image:  13.jpg
tags:   [Python,Dynamics 365,Power BI]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

### 问题

本月，在[Power BI Blog](https://powerbi.microsoft.com/en-us/blog/)网站，增量刷新功能被正式发布，并全面支持Pro用户（此前仅Premium用户可使用）这对于多数PBI报表开发者而言无疑是个好消息，然而，它并不是完美的，对于一些不支持查询折叠的数据源，依然无法实现真正意义上的增量刷新，就如官方文档所述：

><small>大部分支持SQL查询的数据源都支持查询折叠，但一般档案、Blob、Web和OData等数据源通常不支持......在此情况下，交互式引擎在本机补偿并套用筛选，这可能需要从数据源撷取完整的数据集。这会导致累加式重新整理非常缓慢，而且在使用时，该程序可能会用尽Power BI服务或内本地部署数据网关中的资源</small>

因此，对于在报表中使用的来自[Dynamics 365](https://dynamics.microsoft.com/)的数据集，由于其使用[Odata](https://docs.microsoft.com/en-us/odata/overview)协议，显然无法实现真正意义上的增量刷新，而不巧的是，使用Dynamics 365作为数据源的报表项目通常都是企业级的，它的数据集规模通常都比较大，开发者无论是在报表测试还是上传，修改以及发布时，都难以避免因数据庞大而造成的效率低下问题，同时这也容易使该项目的数据刷新占用了本地网关的大量资源，故此问题已成为Power BI报表开发者的痛点之一。

### 思路

事实上，解决此问题的思路是直截了当的。既然直接从Dynamics 365获取的数据集不支持查询折叠，那么我们就把它转换成能够支持查询折叠的数据形式。我们可以写一个程序（本文以Python为例），该程序每天从Dynamics 365上抽取前一天的最新数据，累积存储到关系型数据库中，然后使用Power BI连接该数据库获取该数据，对于Power BI来说，数据源换成了关系型数据库，由于支持查询折叠，一方面数据导入的速度会快很多，另一方面则可以直接利用其内置的增量刷新功能大幅减少每次需要处理的数据量，效率的提升将是成倍的。

*注：对于数据库，本文以SQL Server为例, 但如果您所在的组织使用的是基于云端的Azure SQL Database，那么很幸运，微软提供的[数据汇出服务](https://docs.microsoft.com/en-us/previous-versions/Dynamicscrm-2016/developers-guide/mt788315%28v=crm.8%29?redirectedfrom=MSDN)可以很好的解决与Dynamics 365之间的数据同步问题*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200224151801924.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

也许你会有疑问，为什么非要把数据拖到数据库中进行中转，难道没有别的更简单的方法吗？作为针对企业实施的BI方案，数据存储到数据库是最规范的做法，而且，数据拖过来，别人也可以用，其他的报表工具也可以用，何乐而不为？也许你还有另一个疑问，虽然解决了Power BI增量刷新的问题，但从Dynamics 365到SQL Server的过程中，由于数据源依然是使用Odata协议的，是不是还是要依靠全量刷新？这是个好问题，实际上，本文所述的解决方案是完全增量的，你会在下文中具体了解到利用[Dynamics 365 Web API](https://docs.microsoft.com/es-es/dynamics365/customer-engagement/web-api/about?view=dynamics-ce-odata-9)的查询函数来获取前一日或满足某一特定条件的数据的方法。



### 实现

根据上文思路，我绘制了一个简单的流程图，这也是对实现过程的一个概括：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022515313776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

由于在Power BI上设置增量刷新的方法，官方文档已有详细说明，因此本文不再赘诉，重点将放在前半段：**利用Python脚本从Dynamics 365获取数据，增量更新到SQL Server 数据库中**。

#### 1.登录Azure获取Client ID及Client Secret

Python作为外部程序，要想调取Dynamics 365的数据，需要提供Access Token（获取令牌），而获取Access Token，则不仅需要提供您的Dynamics 365的账户和密码，还需要在Azure Active Directory中创建一个应用，以获取对应的Client ID及Client Secret，首先[登录Azure](https://portal.azure.com)，详细的操作过程可以参考[此文](https://passion4Dynamics.com/2018/12/17/register-Dynamics-crm-app-with-azure-for-oauth-2-0-authentication/)，此处对其做几处关键补充，其一是Client Secret的获取位置参考下图（在value处即为Client Secret）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200224163346574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

其二是，如果你不是您组织的Azure AD管理员，那么你需要获得管理员的许可，在非管理员界面下，API权限分配的选项是灰色的（见下图“Grant admin consent for ```<XXX>```”），因此你需要管理员登录此界面点击此项以为该应用激活API权限。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022416430128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

最后，在后续的代码中还需要利用到OAuth 2.0 token endpoint，你可以参考下图以获取此链接：

*注：实际上即https://login.microsoftonline.com/```<你的租户ID>```/oauth2/v2.0/token*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200224165228421.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

#### 2.利用Python获取访问令牌，以获得访问D365数据的权限

设置好必要的参数，使用posts方法获得访问令牌：

```SQL
#导入必要的库
import requests
import json
import pandas as pd
import numpy as np
from pandas import json_normalize
from datetime import datetime 
from datetime import date

#定义变量
baseurl = 'https://<你的D365的URL>' #以Dynamics.com为后缀
client_id = '<在Azure AD中获取的clientid>' 
client_secret = '在Azure AD中获取的clientsecret'
username = '你的D365用户名' #username
userpassword = '你的D365密码' #password
tokenendpoint = '前文所述的 OAuth 2.0 token endpoint 地址' 
d365_webapi = 'https://<你的D365的URL>/api/data/v9.0'
d365_webapi_table = '/contacts' #此处为你要获取的D365数据的表名，此处以contacts（即客户表）为例
 
#创建令牌请求
tokenpost = {
    'client_id':client_id,
    'client_secret':client_secret,
    'resource':baseurl,
    'username':username,
    'password':userpassword,
    'grant_type':'password'
}
 
token_result = requests.post(tokenendpoint, data=tokenpost)

#获取访问令牌
accesstoken = token_result.json()['access_token']
```

#### 3.获取并转换Dynamics 365数据

拿到Access Token后，就可以获取到D365的数据了，然后针对于Json数据格式，将其转为DataFrame：

```SQL
#构造请求头
d365_post = {
        'Authorization': 'Bearer ' + accesstoken,
        'OData-MaxVersion': '4.0',
        'OData-Version': '4.0',
        'Accept': 'application/json',
        'Content-Type': 'application/json; charset=utf-8',
        'Prefer': 'odata.maxpagesize=500',
        'Prefer': 'odata.include-annotations=OData.Community.Display.V1.FormattedValue'
}

#获取contacts表数据
d365_contacts = requests.get(d365_webapi+d365_webapi_table, headers=d365_post)

#转换数据
dt = d365_contacts.json()
dt = json_normalize(data = dt['value'])
```

#### 4.使用D365 WEB API查询函数

如果前述步骤顺利进行，那么可以使用D365为其WEB API提供的查询函数了，这可以使我们只获取满足我们所设定的过滤条件的数据，从而实现增量抽取。你可以[点此](https://docs.microsoft.com/en-us/Dynamics365/customer-engagement/web-api/queryfunctions?view=Dynamics-ce-odata-9)查看完整的函数列表，本文使用的函数为[Between](https://docs.microsoft.com/en-us/Dynamics365/customer-engagement/web-api/between?view=Dynamics-ce-odata-9)，它允许你只获取某一时间段内的数据。函数的使用方法是在链接上添加参数，你可以事先在浏览器（你已登录Azure的）中进行测试，因此我们只需要在前一个步骤中的请求部分添加这个参数：

```SQL
#导入必要的库
from datetime import timedelta

#设定增量刷新部分的起止时间段，此处设为1天
startdate = (datetime.now()-timedelta(days=1)).strftime('%Y-%m-%d')
enddate = (datetime.now()-timedelta(days=0)).strftime('%Y-%m-%d')

queryfunction = '?$filter=Microsoft.Dynamics.CRM.Between(PropertyName=@p1,PropertyValues=@p2)&@p1=%27new_datecreated%27&@p2=[%27'+startdate+'T00:00:00Z%27,%27'+enddate+'T00:00:00Z%27]'

#获取contacts表数据
d365_contacts = requests.get(d365_webapi+d365_webapi_table+queryfunction, headers=d365_post)

#转换数据
dt = d365_contacts.json()
dt = json_normalize(data = dt['value'])

# 检查数据截断
len(dt['value'])  
```

*注：D365普通企业用户每次调取数据的限制是5000条记录（[PowerApps](https://docs.microsoft.com/en-us/powerapps/powerapps-overview)也为5千，D365专业版为1万），因此需要检查记录数以防发生数据截断，如果您一天的数据已经超过限制也不用担心，使用Between函数是可以以小时为单位的；使用该函数的另一个好处是，它不存在时区上的bug。其他函数，比如[On函数](https://docs.microsoft.com/en-us/Dynamics365/customer-engagement/web-api/on?view=Dynamics-ce-odata-9)，根据我写此文之前的验证，存在时区错误，比如取2020年1月1日的数据，它是以格林尼治时间为准，而非您D365数据集的时区，因此它实际上所取的数据是不准确的，这的确是一个很坑的事实*

#### 5.将D365数据写入数据库

这是Python代码的最后一步了（你可能需要对数据集进行进一步清洗，但这不是本文的范畴），你需要在数据库中创建一个表以存放该数据，最后利用sqlalchemy库将数据写入SQL Server：

```SQL
#导入必要的库
from sqlalchemy import create_engine
import pymssql

#写入数据
engine = create_engine( 'mssql+pymssql://<SQL Server账户>:<SQL Server密码>@<服务器地址>/<数据库名>' )        
result.to_sql(<目标表名称>,engine,if_exists='append')
```

至此，Python脚本完成，接下来只需要设置一个定时作业让该程序在您的服务器上定期执行即可。

### 最后

至此，已实现了数据从Dynamics 365 到SQL Server的增量刷新，成功了一大半，接下来只需要到Power BI Desktop中从数据库导入数据了，你会切身感受到支持查询折叠的数据源的确是比其他数据源传输更快的；更棒的是，在查询编辑器上设定好参数，然后对该数据集设置增量刷新，就可以通过Power BI 本地网关实现从数据库到Power BI Services的增量刷新了：）

*注：关于此部分操作，微软已有[文档说明](https://docs.microsoft.com/en-us/power-bi/service-premium-incremental-refresh)，相信其他博主也会出相关教程，在此不赘诉*

### TIPS

如果您需要将大量的D365历史数据集迁入数据库，而又受到Dynamics 365 WEB API每次取数的行数限制，你的确可以在脚本中使用For循环逐次导入，但此处更推荐使用[DAX Studio](https://daxstudio.org/)，因为在Power BI中调取D365数据没有行数限制（可以说是一种特别照顾），然后你可以使用该工具助你快速一次性将历史数据从PBI数据集全部导入数据库（具体可参考Daniil的[此篇博客](https://xxlbi.com/blog/dax-studio-export-all-data/)）。


