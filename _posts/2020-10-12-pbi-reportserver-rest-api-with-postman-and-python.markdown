---
layout: post
title:  Power BI Report Server REST API 实战
date:   2020-10-12 01:03:50 +0000
image:  16.jpg
tags:   [REST API,PBIRS,Python]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

*本文是D-BI之Power BI REST API系列第四篇，主要讲解利用Postman工具和Python两种方式访问PBIRS REST API*

（封面：木卫一）

### 前述

微软Power BI团队在发布面向Power BI Service的REST API的同时，也发布了针对于其本地版本的Power BI Report Server (PBIRS) 的REST API,  利用它，我们可以用编程的方式在报表服务器创建，删除.PBIX，.RDL以及.XLSX等文件，执行Power BI报表刷新，检查关联数据源连接状态以及查看数据集，数据源详细信息等等，下文将首先使用Postman工具（一种API测试工具）进行API测试，然后使用Python替代Postman完成对应操作。

*（关于Power BI Service版本的REST API使用方法已在此前文章有详细讲述，链接见文末）*

### 实战

面向PBIRS的REST API的详细文档位于[SwaggerHub](https://app.swaggerhub.com/search?type=API&owner=microsoft-rs)网站，该文档
提供了丰富说明，包括：

- CacheRefreshPlans
- CatalogItems
- DataSets
- DataSources
- ExcelWorkbooks
- Extensions
- FavoriteItems
- Folders
- Kpis
- LinkedReports
- Me
- MobileReports
- PowerBIReports
- Reports
- Resources
- Session
- Subscriptions
- System

下文将以获取Power BI报表详细信息为例讲解其调用方法。

##### 一. 使用Postman

首先，登录[Postman](https://web.postman.co/)网站创建新项目. PBIRS API的请求URL格式为：

```SQL
<PBIRS Web门户URL>/api/v2.0/<API命令>
#注意是Web门户URL，而非Web服务URL
```

根据文档，可知要获取Power BI报表数据集的请求URL的API命令为“/PowerBIReports”。了解这些后，只需按下图步骤操作即可：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201012150459579.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

此处需注意在Username部分，考虑到安全因素，可以使用变量避免敏感信息上传服务器，有关在Postman使用变量，参阅[此文档](https://learning.postman.com/docs/sending-requests/variables/)。

完成后直接点击发送，这样我们就成功获取到了Power BI报表的详细信息，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201012153949205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)



##### 二. 使用Python

使用Python调用PBIRS API，过程略不同，原理相通。首先安装必要运行库：

```SQL
pip install requests-ntlm2
```

编辑脚本如下：

```SQL
import requests
import os
from requests_ntlm2 import HttpNtlmAuth

username = <在此输入PBIRS账户>
password = <在此输入PBIRS密码>
baseurl = 'http://localhost/Reports2016/api/v2.0/'
queryurl = 'PowerBIReports'

header = {
    'username':username,
    'password':password,
    'Workstation':<在此输入Hostname>
}

auth=HttpNtlmAuth(username,password)

result = requests.get(os.path.join(baseurl,queryurl),auth=auth)
print(result.text)
```

运行脚本后，成功返回结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020101215554346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

至此，我们成功地使用编程方式调用PBIRS API。

### 总结

根据上文可知，PBIRS API的使用并不复杂，虽然国外相关资料不多，甚至在国内相关资料也几乎空白，但这并不代表PBIRS API没有用处，相反，它的能力被许多人忽视，且还有许多尚待挖掘之处。

### 更多

- [Power BI REST API有多强大？PBI开发者必读](https://d-bi.gitee.io/pbi-rest-api-introduction/)
- [Power BI REST API实战教程：PowerQuery为例](https://d-bi.gitee.io/pbi-rest-api-with-powerquery/)
- [利用Python调用Power BI REST API](https://d-bi.gitee.io/pbi-rest-api-with-python-selenium/)

