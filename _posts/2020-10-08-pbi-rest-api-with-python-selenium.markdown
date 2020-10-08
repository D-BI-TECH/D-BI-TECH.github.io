---
layout: post
title:  利用Python调用Power BI REST API
date:   2020-10-08 01:03:50 +0000
image:  15.jpg
tags:   [REST API,Power BI,Python]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

*本文是D-BI之Power BI REST API系列第三篇，讲解如何利用一段简单的Python脚本实现Power BI REST API的调用，将使用与上文完全不同的方法*

### 前述

上文《[Power BI REST API实战教程：PowerQuery为例](https://d-bi.gitee.io/pbi-rest-api-with-powerquery/)》讲解了PBI API调用的经典方法，而本文将利用简短的Python脚本，更快捷，高效，简易地实现这个效果。在经典方法中，主要问题在于获取Access Token（访问令牌）较为麻烦，需要设置的地方较多，本文将会利用Python，免去自建Azure应用的麻烦，直接借用微软为我们提供的访问令牌来实现PBI API调用。

*注：这里需要厘清一个概念，本文的方法与上文的方法并非PowerQuery与Python的区别，其实上文所述的方法也可以用Python实现，但本文所述的方法并非文档中所述的方法，如上文结尾所述：它不需要创建PBI应用也不需要到Azure做各种麻烦的设置。因此这完全是一篇讲解新技术的文章，不会与前文存在内容雷同*

### 目标

利用PBI API，我们能实现很多功能（具体可阅读[本系列的第一篇](https://d-bi.gitee.io/pbi-rest-api-introduction/)），本文选出以下三个目标作为演示：

1. 实现读取Power BI 所有数据集的详细信息
2. 实现对Power BI某数据集的数据刷新
3. 实现在Power BI创建数据集并向其中的数据表追加一行数据

### 原理

这里的原理简单得说，就是借用微软为我们生成的访问令牌来使用PBI API。首先，我们打开[Power BI REST API文档](https://docs.microsoft.com/en-us/rest/api/power-bi)，选择任意一个方法，如【Get Datasets】, 然后按下图操作。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007162032663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

在使用PBI账户登录后，页面下方会生成API请求预览，其中就包括了最关键的访问令牌：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007162601209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

点击运行后，得到响应代码200，因此我们就成功地调用了PBI API。

![在这里插入图片描述](https://img-blog.csdnimg.cn/202010071629192.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

如下，我们成功得到了所有数据集的信息：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007163420194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

是不是很简单？因此，既然我们可以这么轻松地调用Power BI API, 为什么还要按照经典方法那样，在Azure自建应用以及做各种繁琐的设置呢？我们完全可以把此处的访问令牌拿出来，放到自己的应用中去使用。

然而任何访问令牌都有过期时间，因此我们就需要找到一种方法，可以在每次发送请求前，自动地在文档中完成登录，并且获取最新有效的访问令牌，然后再利用访问令牌，实现数据集信息读取，数据集刷新，报表读取和创建等丰富的操作。完成这些动作恰恰是Python的用武之地，利用[selenium](https://selenium-python.readthedocs.io/)库就可以实现这一点。因此，按照这个思路，我们就“不需要创建PBI应用也不需要到Azure做各种麻烦的设置”，就能够成功使用PBI API！




### 实战

##### 1.安装驱动

本文以谷歌浏览器为例，其他国际主流浏览器皆支持且方法基本相同。由于需要使用selenium，因此要先安装驱动。安装驱动前需要注意您Chrome所对应的版本：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007170452277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

然后点此下载安装包，请注意选择与浏览器相对应的版本，下载完成后解压到任意全英文目录：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100717205870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

##### 2.安装selenium包

直接使用PIP安装即可：

```SQL
pip install selenium
```

##### 3.编写代码&测试

最后，利用以下脚本，即可成功调用PBI API。此处以目标一为例。在下方备注处输入您对应的信息即可运行脚本进行测试。

```SQL
import requests
import selenium.webdriver as sw
import time

account = <此处输入具备读写权限的PowerBI账号>
password = <此处输入对应的PowerBI密码>
queryurl = 'https://api.powerbi.com/v1.0/myorg/datasets' 
#此处输入查询URL，这里以目标一（获取所有数据集）为例

browser = sw.Chrome('C:/Program Files/chromedriver.exe')
#此处输入上文解压的selenium驱动程序路径

browser.get("https://docs.microsoft.com/en-us/rest/api/power-bi/datasets/getdatasets")

browser.find_element_by_css_selector(".action.action-interactive").click()
time.sleep(3)
browser.find_element_by_css_selector(".button.is-primary.is-radiusless").click()
time.sleep(3)
browser.find_element_by_id("i0116").send_keys(account)
time.sleep(3)
browser.find_element_by_id("idSIButton9").click()
time.sleep(3)
browser.find_element_by_id("i0118").send_keys(password)
time.sleep(3)
browser.find_element_by_id("idSIButton9").click()
time.sleep(3)
if browser.current_url == 'https://login.microsoftonline.com/common/login':
    browser.find_element_by_id("idSIButton9").click()
    
time.sleep(6)
header_with_accesstoken = browser.find_element_by_css_selector(".small").text.split("\n",1)[1]
browser.close()

header = dict(Authorization=header_with_accesstoken.split(":",1)[1])
if(header_with_accesstoken!=''):
    header = {'Authorization': header_with_accesstoken.split(":",1)[1].lstrip(" "), 'Content-Type': 'application/json'}
    
queryresult = requests.get(queryurl, headers=header)    
```

脚本执行成功后，我们所有数据集的结果就都包含在了queryresult里。运行queryresult，返回200，表明调用成功，然后运行queryresult.text返回内容。如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100717442995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

至此，测试成功，实现目标一。

*注：若脚本运行时报错no such element: Unable to locate element，则需在time.sleep()增加停顿秒数*

##### 4.实现目标二：数据刷新

利用PBI API来灵活，自动化地执行Power BI数据集刷新任务也是PBI API的主要用法之一。如下图，我们拥有一个名为【DEMO_DBI_BLOG】的数据集，收录了D-BI博客网站的所有文章目录，现在我们需要利用PBI API来执行该数据集的刷新。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201008000014198.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

根据文档，执行数据集刷新的请求URL为：

```SQL
POST https://api.powerbi.com/v1.0/myorg/datasets/{datasetId}/refreshes
```

此处datasetId可在对应数据集的URL上找到。因此在原脚本基础上新增变量：

```SQL
queryurl_refresh = 'https://api.powerbi.com/v1.0/myorg/datasets/<要执行刷新的数据集ID>/refreshes'
```

在原脚本最后一行追加以下代码，执行该数据集刷新，注意此处须使用POST方法：

```SQL
requests.post(queryurl_refresh,headers=header)
```

返回代码202，代表执行成功：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201008003740308.png#pic_center)

回到Power BI Service, 查看其刷新历史记录，如下图，该数据集刷新成功。

*注：由于数据集太小，刷新时间过短，因此刷新了两次以确保无误*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201008003515717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70#pic_center)

从上文可知，要利用PBI API实现其他功能，仅需要使用对应的查询URL以及适当的请求方法即可。对于目标三（实现在Power BI创建数据集并向其中的数据表追加一行数据），
原理也相同，此处不做重复说明，下文演示环节将动态展示整个过程。

### 演示

这是D-BI博客网站近半年的文章目录，现在需要我们利用 REST API向数据集中添加一行数据作为测试, 下方视频展示了完整的实现过程。

[视频: Power BI REST API 实战演示（Python）](https://www.zhihu.com/zvideo/1297479372132405248)





### 后续

至此我们已具备使用PBI API读写Power BI Service的技术能力，无论是第二篇介绍的经典方法，还是本文利用Python演示的【便捷】方法。后续将会发表Power BI REST API系列第四篇，也许会是该系列最后一篇，主要讲解利用PBI API读写基于本地部署的PBIRS的技术，敬请期待！

