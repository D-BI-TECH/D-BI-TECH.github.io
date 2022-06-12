---
layout: post
title:  简谈企业Power BI CI/CD实施框架
date:   2022-06-12 01:03:50 +0000
image:  31.jpg
tags:   [Power BI,DevOps]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 无
---


### 概述

在企业场景中，BI报表更多地作为一项IT服务，而绝不仅仅只是报表工具而已。同理，也正如我此前多次阐明，Power BI是一套服务，绝不仅仅只是Power BI Desktop，它的开发，测试与部署，需要得到有效的管理。因而，无论是基于合规性，还是协作效率等方面的考量，企业实施PBI持续集成与部署，都是百利无害的。

然而，当前国内外的大小企业，已经成功实施PBI CICD的案例并不多，我虽无法完全了解他们的做法，但基于自身的经验和理解，我提出了两种实施思路，供读者参考。

下文我会通过流程图和一些截图来帮助读者理解这种思路，但具体的技术操作此处不会STEP-BY-STEP， 因为官网文档有的，自己能摸索出来的，以及一些我在此前博客中讲过的，没有在这里重复的必要。



### 方案A

这两种方案并不只是理论，他们在实际的技术实施上都是行得通的。其中第一种方案（Plan A）是我个人比较推荐的方案，它使用Onedrive for business做版本控制，用Power Automate做持续部署，具体如下：

![01](https://img-blog.csdnimg.cn/aa51924816a645d6b179fd7d27d5d05e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_30,color_FFFFFF,t_70,g_se,x_16)

这种方案无论是对于拥有单个开发团队的中型企业，还是多个开发团队的大型企业，都是适用的。对于持续集成的部分，其做法即是为开发团队创建一个共享Onedrive，在该Onedrive下可以创建不同的folder以供不同的PBI项目组使用，并将其配置到用于DEV环境的工作区：

![02](https://img-blog.csdnimg.cn/100f41731bbe49828ab08055d694fd64.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_30,color_FFFFFF,t_70,g_se,x_16)

在该模式下，开发者无需将报表部署到工作区，而只需要将其上传到Onedrive即可，其好处如下：

1. 使用Onedirve托管DEV环境中的PBI报表文件，任何版本变动均与该工作区同步
2. 有利于PBI开发团队的管理，分工，以及版本协同
3. 充分利用Onedrive本身具有的版本回退的功能特性，轻松实现PBI文件版本控制

此外，读者也不必担心的是，此模式下PBI数据集与数据源的连接以及刷新是不受影响的，此外，由于我们仅需为DEV工作区配置Onedrive，故而待其后续发布到UAT或生产环境后，Onedirve中的任何版本变动都只会影响DEV环境，不会影响UAT和生产。

持续部署的部分，我们恰好可以利用Power Automate的Onedrive或Sharepoint的触发器，这样，我们可以实现每当一个新的版本上传到Onedrive后，自动触发后续的工作流：

![03](https://img-blog.csdnimg.cn/16092575370143eca07ed599d52a805f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_30,color_FFFFFF,t_70,g_se,x_16)

因此在工作流的后续部分，我们可以调用PBI REST API对Power BI内容进行操作，比如实施部署并刷新对应的数据集：

![04](https://img-blog.csdnimg.cn/27fe33fbb09346f989a687541d769ce4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_30,color_FFFFFF,t_70,g_se,x_16)

*注：关于利用Power Automate调用Power BI REST API，可参考[此文](https://d-bi.gitee.io/pbi-rest-api-supplement/)*


### 方案B

Plan B相对而言则略麻烦一点，它使用Github或Bitbucket等类似平台 (下文以Github代称) 作为PBI文件的托管平台与版本管理工具，而使用Azure DevOps或Jenkins这种专用的CICD工具（下文以DevOps平台统称）进行持续部署。具体如下：

![05](https://img-blog.csdnimg.cn/e71d91d802044f0a894831c3c56c4c4e.png)
在该模式下，开发者将PBI文件Push到Github，并在DevOps平台建立管道运行具体的部署代码来实现CICD，其优势如下：

1. 能够利用DevOps平台丰富的CICD功能
2. 能够与企业其他类型的开发项目在同一个CICD体系下进行管理
3. 能够与企业其他类型的CICD项目（比如一些ETL项目）进行集成

但相比Plan A不足的点在于目前Github不能直接配置到Power BI工作区，无法像方案A那样与DEV环境实时同步，因此多了一个部署到DEV的环节；另一方面则是该方案实施起来要稍微麻烦些。下图是我使用Jenkins实现方案B的部署成功截图 （我在使用Jenkins的测试过程中确实费了一点周折，但具体步骤不在本文范畴内，若有机会另行赘诉）。

![06](https://img-blog.csdnimg.cn/bada0e88f589481ca753c78594c19264.png#pic_center?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_30,color_FFFFFF,t_70,g_se,x_16)

相比之下，使用Azure DevOps做部署的过程则顺利得多。下图T1和T2是我分别用两种不同方法实现Power BI自动部署的成功记录，实际情况下，每当开发者将新版本的PBIX文件推送到Github，则DevOps管道自动被触发，将PBIX自动部署到PBI工作区并完成数据刷新。

![07](https://img-blog.csdnimg.cn/bfaf30b486c9473387bf52256a64ab06.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_RC1CSSB8IERhdmlzIG9uIEJJ,size_30,color_FFFFFF,t_70,g_se,x_16)


### 总结

通常情况下，我更倾向于方案A的模式，它方便，简明，高效，易于管理。但如果PBI的集成与部署需要符合企业级框架下统一的部署规范，或者需要与ETL等其他项目进行集成的需要，那么方案B或许是唯一选择。此外，IT团队实际实施此方案时，要考虑的方面还很多，比如报错处理，版本回退，数据集与数据流间的依赖关系和部署次序等等问题，而本文旨在抛砖引玉，具体的方案则一定是需要团队依据实际情况做相应调整和完善。



***End~***


-----------------

**关注作者： [知乎](https://www.zhihu.com/people/zhang-zhe-hong-01/posts)   [Power BI官方社区](https://community.powerbi.com/t5/user/viewprofilepage/user-id/220984)**
