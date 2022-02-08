---
layout: post
title:  Power BI 企业数据安全 (BYOK与OLS)
date:   2022-02-09 00:00:50 +0000
image:  28.jpg
tags:   [Power BI,Azure Key Vault,Tabular Editor]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 进阶
---

# 前述

数据不仅是企业对资产，也是企业的命脉，其重要性不言而喻。而随着越来越多的企业采用了Power BI作为其商务智能解决方案之一，用户对微软BI体系中数据的安全与合规性的要求也愈加严格。因文章篇幅问题，下文将以PBI及M365管理员的角度，节选Power BI平台中较为核心的数据安全问题，即“数据寄存，数据呈现及数据导出”几个方面进行图文讲解。

# 一、PBI数据安全之数据寄存

当Power BI数据集发布到PBI Service后，其数据实际上存储在Azure Blob Storage, 在默认情况下，数据加密由Microsoft托管，而密钥的值，轮换规则以及加密方式则不受企业或组织的控制。尽管Microsoft使用了足够强大的256位AES加密算法，但在一些企业严苛的信息安全政策及合规部门严谨的审查工作面前，该说辞恐怕无济于事。为了使组织掌控密钥并使数据加密方式与轮换规则符合其政策，在Power BI实施BYOK（Bring your own Key）成为了唯一方案。

## 在PBI实施BYOK

> **注意：**
> 
>  - BYOK目前仅适用于PBI Premium (包括Capacity与Per User)
>  - BYOK主要支持以关系型数据库位数据源的Import数据集    (了解具体限制需参阅[此文档](https://docs.microsoft.com/en-us/power-bi/admin/service-encryption-byok#data-source-and-storage-considerations))


1.登录Azure创建Key Vault, 并分配Unwrap Key及Wrap Key权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/cc3225f7c69346fd8f78f34e2a4e4235.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

2.在Access Policy添加你要用于BYOK的Power BI工作区管理员账户以及Power BI Service服务主体

![在这里插入图片描述](https://img-blog.csdnimg.cn/6ea985987cf543359da2b1bea7142036.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


3.接下来按如下配置创建密钥

![在这里插入图片描述](https://img-blog.csdnimg.cn/79309571380f43e19421b137c34b8a2c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

4.现在，将上一步创建好的密钥应用到PBI容量，目前没有界面可供设置，需要在PowerShell中键入命令

```sql
--安装Cmdlets（已装跳过）
Install-Module -Name MicrosoftPowerBIMgmt
--登录
Connect-PowerBIServiceAccount 
--在整个PBI租户中启用BYOK
Add-PowerBIEncryptionKey -Name '<输入Key名称>' -KeyVaultKeyUri '<输入Key URI>'
--注：其中Key URI可在Azure Key Vault中查询复制
--获取容量ID
Get-PowerBICapacity -Scope Individual
--最后，为该容量设置BYOK
Set-PowerBICapacityEncryptionKey -CapacityId <输入容量ID> -KeyName '<输入Key名称>'
```

至此，BYOK即设置完成，输入以下命令检验：


```sql
Get-PowerBIEncryptionKey
```

如图显示即配置成功：

![在这里插入图片描述](https://img-blog.csdnimg.cn/fd751b7a7d9a44a5991c526013cee56a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

需要注意的点是，目前BYOK的实施是在整个容量层面的，而非Workspace层面的。 一旦为容量设置了BYOK，则该容量下所有的工作区都继承容量的BYOK设置。

6.秘钥轮换，可使用如下命令，若要实现定期轮换，可将其配置到已有应用程序，或直接使用Azure Key Vault 中 "Rotation Policy" 预览功能。

```sql
Switch-PowerBIEncryptionKey -Name '<输入Key名称>' -KeyVaultKeyUri '<输入新的Key URI>'
```

此处需要注意，无论是新版秘钥还是当前秘钥，都必须处于启用状态，否则将遇到如下报错。确保秘钥都启用后，问题解决，如图：

![请添加图片描述](https://img-blog.csdnimg.cn/1c3f4a5898c24d95a68b50f0694ef9c6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16)

# 二、PBI数据安全之数据呈现

数据呈现阶段的安全，顾名思义，是指用户在访问PBI报表时，如何基于不同的自定义角色，给予不同用户或不同组设置对数据不同的访问权限，虽然我们可以在工作区决定是否为某用户授予Viewer角色，但无法控制该用户能看哪些数据以及不能看哪些数据。

幸运的是，在Power BI 导入模型中 ，微软提供了两种数据安全设置，RLS和OLS, 前者即PBI用户所熟悉的行级别安全性，其可以基于角色对不同表不同字段利用DAX定义过滤规则，也可以结合USERPRINCIPALNAME() 动态定义访客权限（由于资料众多，此处不展开）； 而后者OLS (Object Level Security) 则是针对字段本身设置的安全性，它可基于角色控制不同字段（包括字段名）对特定用户是否可见，我们可以形象理解为“列级别安全性”。

## 在PBI实施OLS

与RLS相同，OLS同样要先在PBID创建角色，但具体表或字段可见性的安全控制则需要在外部工具[tabular editor](https://tabulareditor.com)中完成。要对指定角色隐藏字段，只需使用该工具连接到模型后，选择需要设置的表或字段，将OLS中对应角色的值设置为None即可，如图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/1685a10a0d794ac8b9c1e4c3ff509c7d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

以下是关于OLS需要注意的点：

1. 如果对某角色隐藏的表或字段在前端报表中有使用，则该角色查看报表时，与之相关的所有可视化都不可用。

2. 虽然无法针对特定度量值实施OLS，但若度量值所引用的字段包含隐藏的字段，则该度量值对指定角色依然不可见。

下图展示的简单例子很好反映了上述两点。由于对角色“User”在字段“OrderQty”的OLS值设为了None，因此左侧的可视化完全不可用，右侧的表由于未使用该字段，也未使用任何引用该字段的度量值，因此显示不受影响。

![在这里插入图片描述](https://img-blog.csdnimg.cn/782f2bfcd49e4af88fcbb60cdaf1c33c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

由于受底层技术限制，当前PBID还无法做到在同一个可视化内，在隐藏特定字段或值的同时不影响其他字段或值的展现，因此，OLS的应用价值主要体现在了用户自助分析方面。

# 三、PBI数据安全之数据防护

(下文续)



-----------------

**关注作者： [知乎](https://www.zhihu.com/people/zhang-zhe-hong-01/posts)  . [Power BI官方社区](https://community.powerbi.com/t5/user/viewprofilepage/user-id/220984)**
