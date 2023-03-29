# Sease at 搜索解决方案 2022

> 原文：<https://web.archive.org/web/sease.io/2022/10/sease-at-search-solutions-2022.html>

![](img/ca57519623a792ea2ac3557be747a90b.png)

## 搜索解决方案 2022

搜索解决方案是 BCS 信息检索专家组的年度活动，主要关注搜索和信息检索领域的从业者问题。

教程分为全天(5-6 小时，包括休息和午餐)和半天(2-3 小时，包括休息)。教程将于 2022 年 11 月 22 日(星期二)在伦敦的 BCS 办公室和/或网上进行，具体时间视情况而定。

地点: **BCS，英国信息技术特许协会，伦敦，25 Copthall Avenue，EC2R 7BP**
地下日期:【2022 年 11 月 22 日

// our tutorial

## 用 Apache Solr 和开源技术实现神经搜索

22nd November, London

学习排名是机器学习技术与 Apache Solr 的首次集成，允许您使用训练数据来提高搜索结果的排名。
一个限制是，文档必须包含用户在搜索框中键入的关键词才能被检索到(然后被重新排序)。
例如，查询“jaguar”不会检索只包含术语“panthera onca”的文档。
这叫词汇不匹配问题。

神经搜索是一种人工智能技术，它允许搜索引擎到达那些在语义上与用户的信息需求相似的文档，而不必包含那些查询词；它通过深度神经网络和数值向量表示来学习你的集合中的术语和句子的相似度(所以不需要人工同义词！).

这篇演讲探讨了 Apache Solr 9.0 中关于这个主题的第一篇官方文章。
我们从神经搜索的概述开始(别担心，我们保持简单！):我们描述查询和文档的向量表示，以及近似 K 近邻(KNN)向量搜索如何工作。
我们展示了神经搜索如何与深度学习技术(例如，BERT)一起使用或直接在向量数据上使用，以及如何使用 Apache Solr 运行这些类型的搜索！

您将了解:
–近似最近邻(ANN)方法如何工作，重点是分层可导航小世界图(HNSW)
–Apache Lucene 实现如何工作
–Apache Solr 实现如何工作，引入了新的字段类型和查询解析器
–如何运行 KNN 查询，以及如何使用它来重新排列第一阶段的通过
–如何从文本生成向量，并将大型语言模型与 Apache Solr 集成

加入我们，探索这个令人兴奋的 Apache Solr 新特性，并了解如何利用它来改善您的搜索体验！

[More details](https://web.archive.org/web/20221130071853/https://www.bcs.org/membership-and-registrations/member-communities/information-retrieval-specialist-group/conferences-and-events/search-solutions/search-solutions-2022/search-solutions-2022-tutorials/)// TUTORIAL SCHEDULE

9:00–9:20–语义搜索问题介绍(词汇不匹配问题、语义相似性)
9:20–9:40–从文本到矢量(稀疏与密集矢量表示)
9:40–10:10–近似最近邻(ANN)方法的工作原理，重点是分层可导航小世界图(HNSW)
10:10–10:40–Apache Lucene 的工作原理
10:40–11:10–Apache 的工作原理 引入了新的字段类型和查询解析器
11:10–11:30–Break
11:30–12:00–如何运行 KNN 查询以及如何使用它对第一阶段的传递进行重新排序
12:00–12:35–如何从文本中生成向量以及将大型语言模型与 Apache Solr 集成
12:35–13:05–13:20–局限性以及如何减轻这些局限性
13:05–13:20

// slides

[https://web.archive.org/web/20221130071853if_/https://www.slideshare.net/slideshow/embed_code/key/28JV9yw8tIzq3q?hostedIn=slideshare&page=upload](https://web.archive.org/web/20221130071853if_/https://www.slideshare.net/slideshow/embed_code/key/28JV9yw8tIzq3q?hostedIn=slideshare&page=upload)

// your tutor![](img/bd3ac3b1fc6e164c9d89edb8b0e75f06.png)[](https://web.archive.org/web/20221130071853/https://www.linkedin.com/in/alexbenedetti/)*[](https://web.archive.org/web/20221130071853/https://twitter.com/AlexBenedetti)* **#### 亚历山德罗·贝内代蒂

Founder @ Sease
APACHE LUCENE/SOLR COMMITTER
APACHE SOLR PMC MEMBER****// video

[https://web.archive.org/web/20221130071853if_/https://www.youtube.com/embed/DZTaiESWl5o](https://web.archive.org/web/20221130071853if_/https://www.youtube.com/embed/DZTaiESWl5o)**