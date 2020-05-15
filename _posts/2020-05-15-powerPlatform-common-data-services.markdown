---
layout: post
title:  Power Platform：一文理解CDS (含实操)
date:   2020-05-15 03:03:50 +0000
image:  07.jpg
tags:   [Power Platform,PowerApps,CDS]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 入门
---

**(本文主要讲解Power Platform中的Common Data Service, 下文将会依次讲解其概念, 用法以及安全性 etc.)**

### 概念: 什么是CDS?

CDS, 全称Common Data Services(公用数据服务), 按照[文档](https://docs.microsoft.com/zh-cn/powerapps/maker/common-data-service/data-platform-intro)的说法, 有如下几点:

- CDS可以安全地存储和管理业务应用程序使用的数据
- Common Data Service 内的数据存储在一组实体中。 实体是一组用于存储数据的记录，类似于表在数据库中存储数据
- Common Data Service 包括一组覆盖典型情形的标准实体，但是，您还可以创建针对您的组织的自定义实体，并使用 Power Query 用数据填充它们
- 应用制造者随后可以利用 Power Apps 使用此数据生成丰富的应用程序. 

对于不熟悉Power Platform的用户而言, 以上解释很难让人真正理解究竟什么是CDS以及它可以起到什么样的实际作用, 因此在这里有必要通俗一点讲: CDS其实就是一个部署于云端(Dynamics 365)的数据库. 这个数据库有什么特别的地方?  即是其引入了三个重要的概念: **实体**, **实体关系**和**字段**:

- 实体: 通俗讲,实体即是数据库中的表, 但它与一般意义数据库的表**存在异同**. 该表有两种类型, 活动实体和非活动实体, 活动实体是一种较为特殊的表, 用于存储事件(Email, 会议及工作事项等), 因此该实体主要应用于Power Automate, 且其中有系统字段(在该实体创建时默认生成)用于描述事件的状态. 非活动实体即一般意义上的数据表, 你可以在创建该类实体时, 从Excel等数据源将数据导入(这会在下文中介绍操作方法) ; 此外, 实体还因其不同的权限等级被划分为多个级别, 从大到小即: 组织-业务部分-团队或用户-无.
- 实体关系: 顾名思义, 实体关系即CDS数据库中表之间的关系, 分为一对多和多对多两种(多对一本质上也是一对多,只是观察角度不同), 同样地, 和一般意义上的关系型数据库相比, 实体关系又定义了一些不一样的规则, 比如它引入了一个被称为['级联'](https://docs.microsoft.com/zh-cn/powerapps/maker/common-data-service/data-platform-entity-lookup#add-advanced-relationship-behavior)的概念, 这可以使你根据业务需求为不同表之间定义更合适的关系以提高工作效率, 比如在一张表中删除了一条记录,与之相关的其他实体表中与该记录关联的数据是一并删除,还是保留数据仅删除关系, 就可以通过级联的设置定义规则. 因此可以这样定义, **实体关系是能使CDS与业务流程结合得更好的表关系, 这使得CDS的业务性质大于其存储性质**.
- 字段: 实体中的字段即表中的列, 此处没有什么特别需要说明的, 需要注意新建列之前设定好该字段是否为"必要"以及"可搜索"的, 此外, 可以在原实体基础上新建计算列, 此处关于建立计算列需要的函数可参考[此文档](https://docs.microsoft.com/zh-cn/dynamics365/customerengagement/on-premises/customize/define-calculated-fields).

CDS除了能够和你的业务或工作流更好地结合外, 另一大优势是其**对数据资源的整合性**:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514173652847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

Common Data Services可以整合多种数据源, 包括关系型数据库, Excel, Power Platform 数据流等等, 如下图所示:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514174124898.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

此外, CDS还整合了Power Query, 这意味者用户可以像在Power BI那样, 在导入数据之前利用M函数进行基本的数据清洗, 并且可以为数据源设置手动或自动刷新. 数据存放在CDS后, 整个Power平台都能够利用此数据, 构建你的工作流, 应用程式以及交互式报表等等. 因此CDS不仅仅是PowerApps的基础数据平台, 其完全可以作为一个与应用直接集成的**多功能数据共享平台**. 
 
### CDS应用: 初步设定
 
 首先, 由于CDS是随附在PowerApps的, 因此必要的条件是您组织已经购买了PowerApps的License, 或者仅出于学习或演示目的, 可以在新建环境时选择试用30天. 在[PowerApps网站](https://make.powerapps.com/)新建环境后需要创建数据库,如下:
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514175803466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 接下来填写语言和货币种类即可:

 (一个建议: 推荐选择英语, 理由很简单, 日后使用时如果报错, 相关信息以英文显示对你解决问题有很大帮助, 毕竟百度基本解决不了什么问题, 更多的你需要求助于谷歌以及官方社区. 这点我作为一名爱折腾的全栈数据开发深有体会)

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514180142245.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### CDS应用: 导入数据(Excel为例)

 如前文所述, CDS支持丰富的数据源, 关于导入数据这一块, 目前官方文档仅提供了[Odata数据的导入方式](https://docs.microsoft.com/zh-cn/powerapps/maker/common-data-service/data-platform-cds-newentity-pq), 本文将结合实际, 以大家使用最频繁的Excel为例(其他平面文件也类似)导入数据. 首先点击实体, 在上方选择"Get Data",如下所示:

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514181052843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 进入到数据源选择界面后选择Excel, 进入到下图界面. 此处你可能会遇到三个问题, 第一是Browse Onedrive按键为灰色不可用. 这是因为您组织未购买Onedrive for Business或O365, 如果有Office 365则可以直接导入Onedrive上的共享文件. 第二个问题既是下图标红的警告信息, 如图示: 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200514181920346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 这是由于浏览器不允许弹窗的设定, 此处以Chrome浏览器为例, 进入设置--高级(如下图示), 在Pop-ups and redirects处把PowerApps网站加入白名单后刷新浏览器即可解决问题. 
 
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515101746457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 最后一点是你需要将Excel文件里的数据格式化为资料表, 否则Power Query可能无法识别其中的数据, 操作步骤很简单, 可参考[此文档](https://support.office.com/en-us/article/create-and-format-tables-e81aa349-b006-4f8a-9806-5af9df0ac664?ui=en-US&rs=en-US&ad=US). 
 接下来即从你的Onedrive导入你的Excel文件, 你可以在PowerQuery里按需对数据进行清洗:
 
 *(注: 下图数据为恒生指数近10年股价与成交量数据)*

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515102209235.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

 完成后进入下一步,就是设置数据刷新的频率了, 这里不支持直连, 但可以设置**分钟层级**的高频率刷新:
  
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515104002397.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

  创建好刷新计划后, 数据导入部分就完成了, 现在你的Excel数据源就可以定期更新到CDS数据库了. 
  
### CDS应用: 在PowerApps使用CDS数据

  在PowerApps使用CDS数据则十分简单, 可以在创建应用时直接从资料开始:

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515110106677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

  此处奇怪的一点是所有的实体名称都被加上了S后缀, 但没有关系, 建立好连接后即可在PowerApps中使用CDS的数据:

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515111029666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)
  
  当然你也可以直接以空白画布建立应用程序之后, 在view选项卡处连接数据源. 
  
### CDS应用: 在Power BI使用CDS数据
   
   此处需要确保在Power BI Desktop登录的账号具备连接到CDS的权限, 然后在下方界面填写CDS服务器URL:

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515112653447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

   CDS服务器URL可在此处找到:

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051511285961.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)
   
### CDS应用: 在Power Automate使用CDS

利用Power Automate, 用户在CDS内的所有基本操作: 更新, 删除以及插入数据, 都可以用来触发新的事件, 且触发可以定义多条件, 时间以及作用域. 比如当会计科目实体中新增记录时, 在指定时间触发向财务人员发送通知的事件. 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200515115103307.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

此外, 利用Power Automate还可以将其他的工作流事件写入到CDS进行记录, Power Automate拥有丰富的连接器可供利用. 

### 其他: CDS安全性简述

关于安全性,  首先对于整个组织而言, 所有用户都受到[Azure AD](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis)的管理, 因此这里首先有一个域层级的安全边界, 其次则是验证用户是否拥有PowerApps的license. 此外, CDS以不同的环境作为安全边界, 每个环境都拥有独立的CDS数据库, 环境之下有角色, 不同角色拥有不同的权限, 如文档所述: "Common Data Service 使用基于角色的安全性将一组权限组合在一起。 这些安全角色可能直接与用户关联，也可以与 Common Data Service 团队和业务部门关联。 然后，用户可以与团队关联，这样与该团队关联的所有用户都可以利用此角色". 此外, 管理员除了可以将个人或团队分配到某个角色, 还可对于同一个用户可以针对不同的环境配置不同的角色, 这样不同的用户在不同环境的权限都可以有不同的设置, 以此来实现组织内跨部门的数据访问限制(事实上这些设定和SQL Server甚至Windows的安全设置都有很多共同点), 而从数据库的角度看, 不同的实体本身具有不同的权限级别(如前文所言), 强大之处在于, 这种权限控制可以到达字段级别, 关于此可以参考[此文档](https://docs.microsoft.com/zh-cn/power-platform/admin/field-level-security), 此处不做展开. 

### 其他: 关于CDS数据接口(API)

Common Data Service的数据接口与Dynamics 365基本相同, 使用了基于Odata(开放数据协议) 4.0版本的WEB API, 因此你可以使用自己熟悉的语言通过调用该接口获取CDS中的数据, 官方文档提供了[C#版本的使用教程](https://docs.microsoft.com/zh-cn/powerapps/developer/common-data-service/webapi/quick-start-console-app-csharp), 如果你希望使用Python, 可以参考我此前发布的一篇[关于使用Dynamics 365数据接口]({{site.baseurl}}/2020-02-25-dynamics365-incremental-refresh-python/)的文章, 需要注意一定要事先在Azure AD为应用分配好权限. 
