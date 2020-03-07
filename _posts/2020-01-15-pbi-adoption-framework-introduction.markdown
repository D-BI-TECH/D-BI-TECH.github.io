---
layout: post
title:  Power BI Adoption Framework 介绍
date:   2020-01-15 04:03:50 +0000
image:  09.jpg
tags:   [Power BI,数据文化]
author-name: Davis ZHANG
author-image: Davis.jpg
level: 无
---

### 什么是Power BI Adoption Framework

Power BI Adoption Framework（简称AF）是近期引入的一个全新概念，旨在通过一系列包含多个方面的结构化方法，为企业实现以Power BI为核心的商业智能解决方案，最终助力企业形成数据驱动文化。来自[Matthew Roche的博客](https://ssbipolar.com/2019/12/12/the-power-bi-adoption-framework-its-power-bi-af/amp/)很好的概括了它：“**此内容提供了一组指导、实践和资源，以帮助组织构建数据文化、建立Power BI卓越中心并在任何规模上管理Power BI。**”

### 为何要使用Adoption Framework

首先需要说明的是，Adoption Framework，顾名思义即为采用或实施框架，因此它是一个架构，是一种规范性的指引和建议，而并不是一步步的教你如何做，不同的企业，不同的组织有其特定的行业性质和文化环境，因此制定成熟完善的采用实施计划需要具体分析。尽管如此，Adoption Framework仍然具有很大的参考意义，因为它提供了实施BI策略的各个环节中的各个流程，以及可能遇到的风险和相应对策。

### 如何推进企业Power BI的采用与实施

感谢来自印度的BI专家[Manu Kanwarpal](https://www.linkedin.com/in/manukanwarpal/)制作的幻灯片，下图十分简洁明了的介绍了Power BI AF的实施步骤：

1.ENVISION 愿景  
2.ON-BOARD 达成一致  
3.DRIVE VALUE 价值驱动  

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200114224922110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

##### ENVISION 愿景

这部分的核心是项目的执行赞助人是否有建立以Power BI为核心的BI解决方案的决心，高层的赞助是十分关键的，这影响着License规划，用户分配等问题，而你首先需要明确的是，**Power BI不是目的，它只是个工具，利用PBI帮助企业建立数据驱动文化才是真正的目标，最关键的，是心态和行为**。因此必须要向高层证明Power BI的价值。这里有两点，首先，如果组织中已经使用了其他的BI工具，那么要清楚地展示出Power BI的相对优势，至少有以下几点：
1. 拥有强大的Power BI社区 - [Power BI Community](https://community.powerbi.com/)
2. 可以在[Power BI Idea](https://idea.powerBI.com)发起或为他人发起的产品功能改进意见投票，微软Power BI的开发团队一直在关注着，如果某个功能的票数较高，他们就会考虑为该功能的开发进行排期。
3. Power BI Services中灵活而强大的报表分享与权限控制，便于团队的协作开发和共享数据集。
4. Power BI真正实现了敏捷BI而非传统的瀑布开发模式。
5. 每个月都会有产品更新，加入一些新的功能从而提高产品使用体验，比如Power BI Desktop, 你可以在[Power BI Blog](https://powerbi.microsoft.com/en-us/blog/)关注最新的动态。

当然，还有很多其他的功能方面的优势，在此不赘述。但即使如此，你也可能会遇到一些Power BI无法完成的报表项目，毕竟再好的BI工具都有它的局限性，但幸运的是，微软BI最大的优势是它有很多种工具可以很好的集成（如PowerApps, PowerAutomate以及Report Builder等)，**因此实现完整BI的方式是使用一组工具而不是单独使用Power BI**, 顺便提一句，其实这也是本人一直倡导的方式，因此你可以发现在我的博客中不会局限在Power BI这一范畴，而是不断尝试建立Power BI与其他工具的联系与配合。
其次，在明确Power BI优势以后，就需要培养与建立一个先导团队，在此之中可以组织培训，不仅提出解决方案，还要展示具体用例，建议准备好相关的支持文档或视频，内容可以分为具体解决方案以及探索该方案的过程，这样用户可以理解任何挑战，在他们创建解决方案时，他们可以更好的利用这些经验。

##### ON-BOARD 达成一致

这一阶段是举办一个PBI采用研讨会，设计到实施Power BI的各个方面，包括：
1. Governance 管制
2. Services Management 服务管理
3. Security 安全性
4. Rollout ＆Support 部署与支持

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020011500123488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

这里会涉及到很多个方面，比如赞助，是高层赞助还是某个业务部门赞助；又比如分工，谁负责培训，谁负责数据清洗或建模，谁负责连接业务与技术开发合适的数据架构等等；治理方面：如何管理自助式BI文化，如何实现数据民主化；安全性方面：Power BI如何安全地存储您的数据，对于不同的报表和数据集，哪些用户拥有权限，如何分配用户权限等；部署与支持方面：License如何分配, 如何建立内部的BI支持团队等等。此处内容众多，需要会议反复商讨达成一致。

##### DRIVE VALUE 价值驱动

最后这一阶段是大家看到BI产品带来的持续的价值并定期使用。此处引申出了BI部署获得价值从而驱动人们改变行为的四个阶段：
1. Deployment 部署。IT部门在技术上的部署
2. Activation 启动。用户逐渐了解产品功能但需要在实际项目中获得指导
3. Adoption 采用。用户获得产品带来的价值，规律性地采用解决方案
4. Proficiency 精通。用户熟练使用Power BI等产品，充分利用整个BI解决方案，能够用数据创造价值，并改变了他们的业务决策行为。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200115005604241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_d3d3LmQtYmkudGVjaA==,size_16,color_FFFFFF,t_70)

### 总结与拓展

Power BI是一款很好的自助式BI工具，作为一款工具，入门及使用并不难，**真正难的地方在于使用它为业务创造价值，在企业中形成真正的数据驱动文化**，因此，高层的重视，清晰的BI战略，在会议中完善计划、达成共识等等，每一步都至关重要，每一步都并非易事。

本文以微软在Github发布的Power BI AF的PPT为基础进行介绍，原PPT可[点此](https://github.com/pbiaf/powerbiadoption)进入到达Github地址，涉及方面众多，但内容尚不够详实，且本文作为理论性介绍文章，篇幅有限不能尽言，但你还可以在PPT中大致了解关于AF框架的治理、服务管理及安全性方面的知识。最后引用三国曹操的一句注辞："兵无常势，水无常形，临敌变化，不可先传"，学习框架理论固然重要，走出真正属于您组织的Adoption Framework尚需在实战中不断积累经验。
