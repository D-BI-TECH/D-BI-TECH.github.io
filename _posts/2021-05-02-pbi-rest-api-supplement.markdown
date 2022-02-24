---
layout: post
title:  Power BI API调用注意事项 (By Power Automate)
date:   2021-05-02 19:03:50 +0000
image:  07.jpg
tags:   [Power BI,Power Automate]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 无
---


### 前述

本站关于实现Power BI REST API的博文已有许多，包括：

 - [Power BI REST API有多强大？PBI开发者必读](https://d-bi.gitee.io/pbi-rest-api-introduction/)
 - [Power BI REST API实战教程：PowerQuery为例](https://d-bi.gitee.io/pbi-rest-api-with-powerquery/)
 - [利用Python调用Power BI REST API](https://d-bi.gitee.io/pbi-rest-api-with-python-selenium/)
 - [Power BI Report Server REST API 实战](https://d-bi.gitee.io/pbi-reportserver-rest-api-with-postman-and-python/)

这些博文尽可能地覆盖了PBI API的所有具体用法。然而，时间过去已近一年，内容也当有所更新及补充，如前文未涉及企业用户常用的Service Principle方式以及使用Power Automate集成Key Vault的做法等，本文将就此做必要补充。

*注：本文假设读者已阅读过前文或相关MS文档，因此某些具体步骤可能视情况省略。*

### 实战

##### 1.注册App并授予权限时，权限授予后效果不会立即应用

同样，移除权限后，效果也不会立即应用

![aaa](https://img-blog.csdnimg.cn/8385937f50234527a031092c7211789c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

##### 2.出于安全与合规性考量，敏感数据应使用Key Vault并启用安全输出（Secure Outputs）

![在这里插入图片描述](https://img-blog.csdnimg.cn/f59e6145600d45dda182aee5cd0cf575.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)


##### 3.获取Access Token时应使用v2.0 URI

![aaa](https://img-blog.csdnimg.cn/813d97db72c545909b753b579d5d8c5d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)


##### 4.调用REST API时验证方式应与获取Acess Token时保持一致
如果获取Access Token时使用的并非是OAuth方式，则调用API时也不能使用该方式，否则报错

![aaa](https://img-blog.csdnimg.cn/0306f0a622fe4b408fd629bc8836950e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)


### 测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/ccd1e00cbb1841319ae09cc8ddbe2d20.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)


(本文首发于Medium, 部分内容有改动)

-----------------

**关注作者： [知乎](https://www.zhihu.com/people/zhang-zhe-hong-01/posts)   [Power BI官方社区](https://community.powerbi.com/t5/user/viewprofilepage/user-id/220984)**
