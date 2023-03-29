# 墨尔本 Sease 搜索和推荐

> 原文：<https://web.archive.org/web/sease.io/2022/06/sease-at-melbourne-search-and-recommendation.html>

## 墨尔本搜索
及推荐

该组织的目标是让 Meetup 更受墨尔本所有面临类似挑战的搜索和推荐开发者、数据科学家和研究人员的欢迎，同时保持高水平的技术深度。

地点:**墨尔本(在线)**日期:【2022 年 6 月 29 日

![](img/07a96df868cf75fc054ce4a4c9b07a26.png)// our talk

## 使用
Apache Solr 神经搜索进行密集检索

神经搜索是从神经信息检索的学术领域衍生出来的一个产业。我们越来越频繁地听说人工智能(AI)如何渗透到我们生活的方方面面，这也包括软件工程和信息检索。

特别是**深度学习**的问世，引入了深度**神经网络**的使用，来解决无法简单通过算法解决的复杂问题。**深度学习**可用于生成信息语料库中查询和文档的向量表示。**搜索**通常包括执行**四个主要步骤**:
–生成描述信息需求的查询表示法–生成捕获其中包含的信息的文档表示法
–匹配查询和来自信息语料库的文档表示法
–为每个匹配的文档分配一个分数，以便根据结果中的相关性建立有意义的文档排名。

通过神经搜索模块，Apache Solr 引入了对基于神经网络的技术的支持，这些技术可以改进搜索的这四个方面。

本次演讲探讨了 Apache Solr 9.0(2022 年 5 月)提供的神经搜索功能的**首次官方贡献:**近似 K 近邻向量搜索，用于匹配和排序。****

您将了解:
–近似最近邻(ANN)方法如何工作，重点是分层可导航小世界图(HNSW)
–Apache Lucene 实现如何工作
–Apache Solr 实现如何工作，引入了新的字段类型和查询解析器
–如何运行 KNN 查询，以及如何使用它来重新排列第一阶段通过加入我们，一起探索这一新的 Apache Solr 特性！

// slides

[https://web.archive.org/web/20221202224726if_/https://www.slideshare.net/slideshow/embed_code/key/eu6b98S9GDYunP?hostedIn=slideshare&page=upload](https://web.archive.org/web/20221202224726if_/https://www.slideshare.net/slideshow/embed_code/key/eu6b98S9GDYunP?hostedIn=slideshare&page=upload)

// our speaker![](img/1257192865321c06443c651935c924e8.png)[](https://web.archive.org/web/20221202224726/https://twitter.com/AlexBenedetti)*[](https://web.archive.org/web/20221202224726/https://www.linkedin.com/in/alexbenedetti/)* **#### 亚历山德罗·贝内代蒂

Founder @ Sease
APACHE LUCENE/SOLR COMMITTER
APACHE SOLR PMC MEMBER****// video

[https://web.archive.org/web/20221202224726if_/https://www.youtube.com/embed/DZTaiESWl5o](https://web.archive.org/web/20221202224726if_/https://www.youtube.com/embed/DZTaiESWl5o)**