# ECIR 2022 赛季！

> 原文：<https://web.archive.org/web/sease.io/2022/05/sease-at-ecir-2022.html>

## ECIR 2022

ECIR 的特色是全论文和海报展示、系统演示、教程、研讨会，这是一个面向行业的活动，传统上非常注重早期职业研究人员的积极参与。

地点:**挪威斯塔万格**
日期:【2022 年 4 月 10 日至 14 日

![](img/40da2c2a8a6914e099aea1e80130254b.png)// our first talk

## 使用
Apache Solr 神经搜索进行密集检索

神经搜索是从神经信息检索的学术领域衍生出来的一个产业。我们越来越频繁地听说人工智能(AI)如何渗透到我们生活的方方面面，这也包括软件工程和信息检索。特别是，深度学习的出现引入了深度神经网络的使用，以解决无法简单通过算法解决的复杂问题。例如，深度学习可以用于产生查询和信息语料库中的文档的向量表示。一般来说，搜索包括执行四个主要步骤:

*   生成描述信息需求的查询表示
*   生成文档的表示，以捕获其中包含的信息
*   将查询与来自信息语料库的文档表示进行匹配
*   为每个匹配的文档分配一个分数，以便根据结果中的相关性建立一个有意义的文档排名

通过神经搜索模块，Apache Solr 引入了对基于神经网络的技术的支持，这些技术可以改进搜索的这四个方面。这个演讲探讨了 Apache Solr 9.1 在 2022 年第一季度对神经搜索功能的第一个官方贡献:用于匹配和排序的近似 K-最近邻向量搜索。

您将了解到:

*   近似最近邻(ANN)方法如何工作，重点是分层可导航小世界图(HNSW)
*   Apache Lucene 实现如何工作
*   Apache Solr 实现如何工作，引入了新的字段类型和查询解析器
*   如何运行 KNN 查询，以及如何使用它来重新排名第一阶段的通行证
*   性能基准与经典 BM25 词汇检索和排序相比如何

![](img/cd430b0e9411a950225aabf0b2e404e4.png)// our speakers![](img/e808bf0ee11cad7c97afa870c612fa31.png)[](https://web.archive.org/web/20221202221431/https://twitter.com/AlexBenedetti)*[](https://web.archive.org/web/20221202221431/https://www.linkedin.com/in/alexbenedetti/)* **#### 亚历山德罗·贝尼代蒂

Founder @Sease
APACHE LUCENE/SOLR COMMITTER
APACHE SOLR PMC MEMBER****![](img/aabac9952f28f2f86d1af68b0a7f8b7b.png)[](https://web.archive.org/web/20221202221431/https://twitter.com/eliaporciani)*[](https://web.archive.org/web/20221202221431/https://it.linkedin.com/in/elia-porciani-89730866)* **#### [埃利亚·波恰尼](https://web.archive.org/web/20221202221431/https://sease.io/elia-porciani)

R&D; SOFTWARE ENGINEER @Sease
SEARCH CONSULTANT********// our second talk

## 评估生产中的排名模型:
对线下和线上体验的看法

评价在信息检索领域起着关键作用。研究人员和从业者设计和开发排序模型来表示用户表达的信息需求(查询)和来自可用资源(语料库)的信息(搜索结果)之间的关系。要验证任何关于排名创新的研究论文，测试

通过比较它们的结果和计算相关性度量来生成模型

确定的基本事实(判断)。

在行业中会发生什么，真实用户与系统交互，商业利益影响相关性的概念，预定义的相关性判断不可用？这个演讲展示了不同领域的公司是如何解决这个问题的

实施离线和在线测试/监控解决方案。对于每个现实世界的应用，本演示描述了:

*   如何处理和实施(A/B 测试、交错、统计显著性计算……)
*   如何收集隐式/显式反馈并用于评估相关性(内部专家团队、用户与系统的交互、收入/利润信号、赞助结果……)
*   实验是如何设计和计划的(当时比较多少个模型，在同一测试中比较哪些模型，如何测试移动/桌面/平板电脑平台……)
*   使用什么开源技术来促进这些任务
*   最常见的陷阱和减少陷阱的解决方案

![](img/e9c82f5a66ae6cd64be254d3c2a128cf.png)// our speakers![](img/e808bf0ee11cad7c97afa870c612fa31.png)[](https://web.archive.org/web/20221202221431/https://twitter.com/AlexBenedetti)*[](https://web.archive.org/web/20221202221431/https://www.linkedin.com/in/alexbenedetti/)* **#### 亚历山德罗·贝内代蒂

Founder @ Sease
APACHE LUCENE/SOLR COMMITTER
APACHE SOLR PMC MEMBER****![](img/be6e4466c31d3f2b810cc341e088e983.png)[](https://web.archive.org/web/20221202221431/https://twitter.com/Anna900433881)*[](https://web.archive.org/web/20221202221431/https://it.linkedin.com/in/anna-ruggero-482902153)* **#### [安娜·鲁杰罗](https://web.archive.org/web/20221202221431/https://sease.io/anna-ruggero)

R&D; SOFTWARE ENGINEER @Sease
SEARCH CONSULTANT********