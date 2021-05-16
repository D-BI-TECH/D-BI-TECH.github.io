---
layout: post
title:  使用Power BI Cmdlets部署|迁移报表
date:   2021-05-16 01:03:50 +0000
image:  27.jpg
tags:   [Power BI,PowerShell]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

*本文实际借迁移报表的需求为例讲解Cmdlets命令用法，属进阶内容，但实操极易*

### 需求
在企业BI场景下，我们也许会遇到一种需求，即使用自动化方式将报表或数据集从一个Workspace（工作区）迁移到另一个Workspace。

### 场景
对于仅使用Pro License的组织，它们希望通过这种方式来模拟PBI管道部署，而对于拥有Premium License (无论是企业版还是PU版) 的组织，通常而言直接使用Deployment Pipeline即可，但依然有少数组织，尤其是对于合规性和安全性管控极严的企业，Deployment Pipeline有不能满足其需求的情况（此处略），而手动发布显然不是明智的做法，但利用Power BI Cmdlets命令，却可完美解决此问题。

*注：Power BI Cmdlets是PowerShell的Modules之一, 其允许Power BI管理员以命令行形式管理Power BI Workspace并与其交互，包括为获取和更改工作区信息，为指定工作区增减用户，指派角色，新建或删除工作区内报表等，其本质上是利用了REST API，但它比REST API更简洁易用*

### 思路
依据需求，案例情况及方案思路如下：
账号1是部署管道的管理员，负责将项目从开发环境（D-BI-C1）发布到准测试环境 (D-BI-C1[test])，账号2是工作区D-BI-TEST的管理员，并具有对D-BI-C1[test]的访问权限。建立一个程序，每当其检测到Deployment Pipeline对指定的数据集发布完成，即验证D-BI-C1[test]指定内容是否已更新，即执行迁移命令。流程图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517001804941.png)


### 实践
首先，根据[文档](https://docs.microsoft.com/en-us/powershell/power-bi/overview?view=powerbi-ps)安装必要的模块。然后以管理员模式运行PowerShell，跟随文档输入命令登录具备Power BI管理员的账号。依据以上思路，迁移报表，分两部分：

- 检测迁出工作区的更新状态
- 若迁出工作区更新，执行报表的导出与上传

其中第一部分需要使用如下命令获取工作区所有（或指定）数据集的更新时间：

```
Get-PowerBIImport -Scope Individual -WorkspaceId  <your workspace ID>
```

第二部分则需要用到两个命令：

- [Export-PowerBIReport](https://docs.microsoft.com/en-us/powershell/module/microsoftpowerbimgmt.reports/export-powerbireport?view=powerbi-ps)
- [New-PowerBIReport](https://docs.microsoft.com/en-us/powershell/module/microsoftpowerbimgmt.reports/new-powerbireport?view=powerbi-ps)

另一种办法是直接使用Copy命令，本文不作赘述。

- [Copy-PowerBIReport](https://docs.microsoft.com/en-us/powershell/module/microsoftpowerbimgmt.reports/copy-powerbireport?view=powerbi-ps)

以下仅展示第二部分流程。如下，内容从开发环境发布到准测试环境

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517002813412.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)
为验证效果，此处使用了参数规则，对于开发环境内容仅取数据集前100行，发布到准测试环境后则取完整数据集：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517003315276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)
执行导出命令

```
Export-PowerBIReport -Id <Input your report ID> -WorkspaceId <workspace ID> -OutFile .\Customer_Test.pbix
```

报表被导出至指定目录下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517003416558.png)

然后上传至测试环境工作区（D-BI-TEST）

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051700371194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)
至此，自动迁移完成：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051700380490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)
需要注意的是，由于迁出工作区具备Premium License (或PPU) , 目标工作区也要求拥有同样的License，否则打开报表将报错：

```
This operation is not allowed...Please try again later or contact support...
```

### 验证

报表显示了完整的数据集，而非仅100行，迁移成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210517004558166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_16,color_FFFFFF,t_70)


### 其他

由于本人今年的个人时间规划，在PBI等技术领域的投入被迫减少，每季度发文量可能会减少至去年三分之一，这种状况可能会持续半年以上，但我依然会持续关注Power BI乃至整个Power Platform的技术动向，与微软产品组，其他MVP或爱好者，通过邮件或Power BI社区网站等方式交流技术问题，感谢本站读者支持。

-----------------

**关注作者： [知乎](https://www.zhihu.com/people/zhang-zhe-hong-01/posts)  | [Power BI官方社区](https://community.powerbi.com/t5/user/viewprofilepage/user-id/220984)**
