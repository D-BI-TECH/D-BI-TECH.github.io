---
layout: post
title:  PBI Report Server部署与配置详解
date:   2019-11-18 06:03:50 +0000
image:  09.jpg
tags:   [PBI Report Server,SSRS]
author-name: Davis ZHANG
author-image: Davis.jpg
---

*【前述】关于Power BI 报表服务器的安装与配置，国内外的教程有很多，但较为全面的介绍却少之又少。通常来讲，你可能只需要完成安装并做简单配置就能正常使用报表服务器，那么你只需要参考[此Microsoft文档](https://docs.microsoft.com/en-us/power-bi/report-server/install-report-server)就已经足够了， 但如果你想详细了解服务器配置的各个方面，本文将会是一个很好的补充。*

一 . 关于Report Server Configuration Manager
-----

Power BI Report Server Configuration Manager 是用于配置Power BI 报表服务器的专用工具，随Power BI Report Server附带安装。由于PBI Report Server是基于SQL Server 2017 Reporting Services 开发而来，PBI Report Server Configuration Manager同样也是基于SSRS 2017 Configuration Manager 进行的定制版，主要区别在于它可以创建面向Power BI报表服务器的报表服务器数据库，大多数功能并无区别。使用它可以使你配置报表服务器的服务账户，数据库，站点URL以及服务邮箱等等。此外，Configuration Manager具有版本局限性，如前所说，PBI Report Server Configuration Manager是基于2017版的，因此最好不要拿该版本的配置管理器去配置版本低于2017的Reporting Services, 同样也不推荐在同一台机器运行不同版本的报表服务器，因为这极易引发一些问题，比如下图中的报错（此情况下，按照官方文档的说法，你需要用对应版本的配置管理器分别配置对应的服务器实例）。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180010863.png)

Power BI Report Server有单点部署和拓展部署之分，拓展部署指多个报表服务器实例共用同一个报表服务器数据库，此情况下，Configuration Manager只会影响其所配置的报表服务器，而非全部。

*备注 1：用于PBI的报表服务器不能和SSRS共享报表服务器的数据库*

*备注 2：如果在生产环境中部署Power BI Report Server，出于安全考虑，强烈推荐将其部署在域环境*

二. 配置服务账户与执行账户
-----

配置服务账户。服务账户已经在报表服务器安装向导过程中预置，你可以在Configuration Manager中根据你的实际情况修改。服务账户主要分以下几种，并作对应说明：

>- 内置-虚拟服务账户  
  <small>虚拟服务账户是具有域网络登录权限的内置最低权限账户。虚拟服务账户之所以称之为“虚拟”，是因为它是一个伪账户，你不能用它来登陆，也不存在密码，但它有足够的权限且可以在后台执行所有必要的操作，当然任何强制密码过期策略对该账户都是无效的，因此它可以规避因此导致的服务中断。我推荐你在测试时使用该服务账户。你可以在【任务管理器-服务-打开服务】中看到它，正式名称为：NT SERVICE\ <SERVICENAME><small>

>- 内置-网络服务  
 <small>同样是一种伪账户，但相比虚拟服务账户权限更多，可访问网络资源，而不仅仅是域网络。不过正由于此，该账户具有一定的安全隐患，所以微软的文档建议我们尽量减少在同一账户下运行的其他服务的数量，因为任何一个应用程序的安全漏洞都会损害在同一账户下运行的所有其他应用程序的安全。你可以在【任务管理器-服务-打开服务】中看到它，正式名称为：NT AUTHORITY\ <SERVICENAME><small>

>- 域账户  
  <small>从安全角度考虑，域用户账户是最受推荐的（使用域用户账户会将报表服务器与其他程序隔离开）。当然前提是你在域环境中，并且你的域windows账户具有必要的权限。如果你决定使用该类别的服务账户，那么最好在“本地用户和组”处勾选“密码永不过期”。名称为：<Domain Name>\<User Name><small>

*备注：修改服务账户通常不会影响报表服务器运行，因为旧账户不会被自动移除。应用修改后报表服务器会自动重启*

配置执行账户。服务账户是用于报表服务器运行的账户，而执行账户则完全不同，它是用于获取外部数据源的账户。比如你需要从远程计算机获取产品图片，且数据不能通过匿名访问获取，那么至少要求该账户在该计算机具有只读权限。每当报表中引用了远程服务器的数据，报表服务器就会在该报表（分页报表或PBI报表）每次刷新数据时，调用该执行账户访问远程服务器。出于安全性考虑，执行账户不要和服务账户相同，并且需要使用域账户(或工作组账户)。你还可以使用rsconfig Utility配置执行账户。

*备注：如果你的计算机使用Microsoft账户登录，需要切换到本地帐户模式*

三. Web服务URL与Web门户URL
-----

配置Web服务URL。配置Web服务URL以使得我们可以在本地网络通过URL访问报表服务器实例，你需要指定虚拟目录名称（最好用英文）, IP地址，TCP端口。IP地址通常使用默认设置即可，TCP端口需要指定一个未被占用的端口，否则会造成报表服务器无法访问。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180052673.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

关于HTTPS端口。HTTPS证书及端口是可选项，如果你希望使用SSL在报表服务器和Web服务之间使用加密的数据传输方式，那么你需要安装一个SSL证书，并指定HTTPS端口（默认为443），然后将证书绑定到URL。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180102954.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*备注：关于安装SSL证书可参考[此文](https://blog.csdn.net/qq_44794714/article/details/103077575)或[此Microsoft文档](https://docs.microsoft.com/en-us/sql/reporting-services/security/configure-ssl-connections-on-a-native-mode-report-server?view=sql-server-ver15)*

配置Web门户URL。Web门户URL即PBI Report Server站点URL，用户通过此地址访问报表服务器站点，上传，下载或查看报表。每当你修改了Web服务URL，一定要到重新应用Web门户URL，否则可能会出现"Web服务URL与Web门户URL不一致"的错误。此外，你可以通过指定不同的主机名和端口指定多个Web门户URL和Web服务URL，比如当你希望内部人员和外部人员访问报表服务器门户时使用不同的安全验证方法，就可以利用此特性，但对于同一实例，虚拟目录只能有一个。
备注：PBI Report Server Web门户URL可以传参（尽管可用的功能暂时不如SSRS及Power BI Services丰富）。比如报表URL+?rs:embed=true可以让报表全屏显示（分页报表及PBI报表），也可以设定filter参数预先切片（仅支持PBI报表），详见[Microsoft文档](https://docs.microsoft.com/en-us/power-bi/service-url-filters)。

四. 配置数据库
------

报表服务器本身没有数据库，而是把服务器上的数据（如共享数据集，Power BI报表以及用户角色等）托管到SQL SERVER。因此它需要管理员在配置管理器将服务器连接到Report Server数据库实例，如果不存在该数据库就可以选择新建，你只需要设置好数据库名称和语言即可，剩下的操作，包括创建特定表，视图以及存储过程等繁杂工作，Configuration Manager会自动帮你完成。

关于连接到数据库的凭据分为三种，见下方说明：

>- 服务凭据  
  <small>该凭据是默认的选项，它使用内置的Windows服务提供·连接到数据库的凭据，具备所需的最低权限。在域环境或用于测试时，如果报表服务器数据库和报表服务器在同一台机器运行，推荐使用此设置。如果数据库和服务器不在同一个域，除非你使用工作组安全设置并且它们在同一个工作组，否则不能使用此设置。<small>

>- Windows凭据  
 <small>使用Windows账户作为凭据。当报表服务器数据库和报表服务器在同一个域环境中时，推荐使用Windows域用户账户作为凭据，但需要注意的是，该域用户必须具备在报表服务器数据库中读写及创建表的权限，否则你将无法成功完成报表服务器数据库配置向导。<small>

>- SQL Sever凭据  
  <small>仅当报表服务器数据库和报表服务器不在同一域环境中用此，因为该情况下你不能使用服务凭据，并且在域环境外的Windows凭据不会有所需的权限，因此只能使用SQL Server账户作为凭据，需要注意该类凭据可能存在安全隐患，并且需要检查该账户是否实施了强制密码过期或“用户在下次登录必须更改密码“，最好取消这些策略以避免不必要的麻烦。<small>

*备注：数据库用户权限以及其他高级设置需要使用SSMS进行设置*

当报表服务器数据库部署于同一域环境下的另一台计算机时，除设置必要的用户权限外，你可能还需要完成以下几个步骤：

1.到SSMS检查数据库实例是否允许远程连接

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180114885.png)

2.开启SQL Server配置管理器，在SQL Server网络配置处启用TCP/IP，并设置其TCP端口：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180123651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

3.在服务器端的控制面板中打开防火墙进入高级设置，新建两个入站规则，分别为TCP 1433 和 UDP 1434。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180128782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

经过这些设置，就可以保证本地报表服务器能够正常连接到远程数据库了。

五. 电子邮件服务
-----

配置电子邮件--准备工作。在PBI Report Server中，可以创建报表订阅，使特定报表（暂支持分页报表）可以按期以你所设定的文件格式发送到指定一个或多个用户的邮箱。在未配置电子邮件前，报表服务器的订阅方式中"电子邮件"是不可见的，配置好电子邮件地址和SMTP服务器后，重启Report Server即可在订阅页面看到"电子邮件"的订阅方式：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180142787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

由于报表服务器订阅服务托管在SQL Server中执行，因此需要开启SQL Sever 代理服务：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180157788.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

配置电子邮件--填写基本信息。在Configuration Manager的"电子邮件设置"处填写发件人地址，SMTP服务器等信息（其中SMTP服务器可以使用远程服务器或本地服务器，此处以使用远程SMTP服务器为例）。根据我的测试，不是所有的SMTP服务器都能连接或验证成功，以下列出我的测试结果供大家参考：

<div class="table-container">
  <table>
    <tr><th>SMTP 服务器</th><th>Outlook</th><th>移动 139</th><th>新浪 Sina</th><th>QQ</th></tr>
    <tr><td>SMTP 地址：</td><td>smtp-mail.outlook.com</td><td>smtp.139.com</td><td>smtp.sina.com</td><td>smtp.qq.com</td></tr>
    <tr><td>测试结果：</td><td>成功</td><td>连接失败</td><td>验证失败</td><td>成功</td></tr>
  </table>
</div>

在身份验证处，选择"用户名和密码(基本)"，用户名即发件人地址，密码处如果是使用QQ邮箱则需要填写授权码，而非你的QQ邮箱密码，当然在此之前你需要到【QQ邮箱--设置--账户】处开启SMTP服务（有关获取QQ邮箱授权码[点此](https://service.mail.qq.com/cgi-bin/help?subtype=1&id=28&no=1001256)查看详情）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180212624.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*备注：如果你指定的SMTP支持匿名连接，你可以选择"无身份验证"，或者你选择使用本地SMTP服务器时，可以选择NTLM身份验证*

配置电子邮件--使用配置文件配置。你也可以打开rsreportserver.config文件填写并修改部分配置 （文件位置: <你的PBI Report Server 根目录>/PBIRS/ReportServer）。其中，在<SMTPServerPort>处填写端口号"25",<SMTPUseSSL>处一般填写False，在<DefaultHostName>填写SMTP服务器的IP地址（但这不是必须的）。
在完成以上步骤后，回到PBI Report Server门户网站，创建一个订阅并设定计划，如果仅用于测试则可以跳过此步（设定计划同样适用于文件共享服务）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180226101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

建立订阅完成后，点击"立即运行"，测试成功后如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180233945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

你所设定的收件人将会成功收到报表订阅邮件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180241709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

六. 文件共享服务
-----

设定文件共享服务，可以在域环境或工作组环境下，让你所授权的用户到共享文件夹查看或获取报表。访问共享文件夹的凭据有两种，Windows凭据和文件共享账户凭据，如果你使用Windows凭据，必须确保该账户拥有对该文件夹的读写权限，如果你使用文件共享账户凭据，则必须事先在Configuration Manager的"订阅设置"处配置文件共享账户，否则该凭据不可用（如下）：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180251564.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

*备注：文件共享账户可以和执行账户相同，但不能和服务账户相同。但此情况下需要确保执行账户的管理员权限，否则会引发" 不允许所请求的注册表访问权"的错误，如果不能得到足够的权限，可以考虑删除执行账户。*

回到PBI Report Server站点，新建文件共享订阅，在下图页面填上必要的信息。此处有一个很好的特性就是"覆盖选项", 你可以选择"添加更新的版本时文件名递增", 这对用户查阅历史数据提供了很大的便利。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180259948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

运行该文件共享订阅，测试成功后如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191201180311850.png)

同时在共享文件夹你会看到报表服务器发送的报表文件：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019120118031721.png)

七. 其他设置
-----

到此，Power BI报表服务器的主要配置已经讲完，如果在配置以上内容时遇到错误，你可以及时查阅报表服务器日志以了解错误原因。此外，还有一些其他的配置也许也是你需要用到的，比如在Configuration Manager中的加密密钥和集成云服务，以及在本文开头提到的拓展部署等等，也许这些内容我会在日后的博客中再做补充。

*End~ :D*