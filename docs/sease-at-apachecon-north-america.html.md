# 北美阿帕契康的季节

> 原文：<https://web.archive.org/web/sease.io/2022/07/sease-at-apachecon-north-america.html>

![](img/7a7dd1af08c138834ca34ada007026e1.png)

## 阿帕契肯
北美

ApacheCon 通过详细的会议、实践研讨会和独立的 Apache 项目跟踪，将用户与最大的全球 Apache 项目社区集合在一起。

地点:**新奥尔良运河街**日期:【2022 年 10 月 3 日-6 日

// our talk

## 神经搜索来了 Apache Solr:
近似最近邻，
BERT 等等！

6th October, Sheraton

学习排名是机器学习技术与 Apache Solr 的首次集成，允许您使用训练数据来提高搜索结果的排名。

一个限制是，文档必须包含用户在搜索框中键入的关键字，以便被检索(然后重新排序)。例如，查询“jaguar”不会检索只包含术语“panthera onca”的文档。这就是所谓的词汇不匹配问题。

神经搜索是一种人工智能技术，它允许搜索引擎到达那些在语义上与用户的信息需求相似的文档，而不必包含那些查询词；它通过深度神经网络和数值向量表示来学习你的集合中的术语和句子的相似度(所以不需要人工同义词！).

这篇演讲探讨了 Apache Solr 9.0 中关于这个主题的第一篇官方文章。

我们从神经搜索的概述开始(别担心，我们保持简单！):我们描述查询和文档的向量表示，以及近似 K 近邻(KNN)向量搜索如何工作。我们展示了神经搜索如何与深度学习技术(例如，BERT)一起使用或直接用于向量数据，以及我们如何在 Apache Solr 中实现该功能，并给出了使用示例！

加入我们，探索这个令人兴奋的 Apache Solr 新特性，并了解如何利用它来改善您的搜索体验！

// slides

[https://web.archive.org/web/20221202234202if_/https://www.slideshare.net/slideshow/embed_code/key/28JV9yw8tIzq3q?hostedIn=slideshare&page=upload](https://web.archive.org/web/20221202234202if_/https://www.slideshare.net/slideshow/embed_code/key/28JV9yw8tIzq3q?hostedIn=slideshare&page=upload)

// our speaker![](img/11830ee464c6a94c64d2a3d0e08e7254.png)[](https://web.archive.org/web/20221202234202/https://www.linkedin.com/in/alexbenedetti/)*[](https://web.archive.org/web/20221202234202/https://twitter.com/AlexBenedetti)* **#### 亚历山德罗·贝内代蒂

Founder @ Sease
APACHE LUCENE/SOLR COMMITTER
APACHE SOLR PMC MEMBER****// video

[https://web.archive.org/web/20221202234202if_/https://www.youtube.com/embed/DZTaiESWl5o](https://web.archive.org/web/20221202234202if_/https://www.youtube.com/embed/DZTaiESWl5o)**