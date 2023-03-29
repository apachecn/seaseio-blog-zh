# 柏林流行语 2022

> 原文：<https://web.archive.org/web/sease.io/2022/05/sease-at-berlin-buzzwords-2022.html>

![](img/942d00ef4c88d4ce1c72492dd22318e7.png)

## 柏林流行语
现场直播

德国关于存储、处理、流式传输和搜索大量数字数据的最激动人心的会议，重点是开源软件项目。

地点: **Kulturbrauerei，**柏林 日期:【2022 年 6 月 12 日-14 日

// our first talk

## 神经搜索来了 Apache Solr:
近似最近邻，
BERT 和 More(流行语)！

13th June, 4:00–4:40 (GMT+2), Kesselhaus

机器学习技术与搜索的第一次集成允许提高搜索结果的排名(学习排名)——但一直以来的一个限制是，文档必须包含用户在搜索框中键入的关键词，才能被检索到。例如，查询“tiger”不会检索只包含术语“panthera tigris”的文档。这被称为词汇不匹配问题，多年来已经通过查询和文档扩展方法得到缓解。

神经搜索是一种人工智能技术，它允许搜索引擎到达与用户的查询语义相似的那些文档，而不一定包含那些术语；它通过利用深度神经网络和数字向量表示，自动学习您收集的术语和句子的相似性，从而避免了对长同义词列表的需要。

这篇演讲探讨了 Apache Solr 9.0 中关于这个主题的第一篇官方文章。

在演讲中，我们将给出神经搜索的概述:我们将描述查询和文档的向量表示，以及近似 K 近邻(KNN)向量搜索是如何工作的。

我们将展示神经搜索如何与深度学习技术(例如，BERT)一起使用或直接用于向量数据，以及我们如何在 Apache Solr 中实现这一功能。

// slides

[https://web.archive.org/web/20221130065156if_/https://www.slideshare.net/slideshow/embed_code/key/rUNFj2cCIUOtgJ?hostedIn=slideshare&page=upload](https://web.archive.org/web/20221130065156if_/https://www.slideshare.net/slideshow/embed_code/key/rUNFj2cCIUOtgJ?hostedIn=slideshare&page=upload)

// our speaker![](img/6b9782039596312eb93c84408b895176.png)[](https://web.archive.org/web/20221130065156/https://twitter.com/AlexBenedetti)*[](https://web.archive.org/web/20221130065156/https://www.linkedin.com/in/alexbenedetti/)* **#### 亚历山德罗·贝内代蒂

Founder @ Sease
APACHE LUCENE/SOLR COMMITTER
APACHE SOLR PMC MEMBER****// video

[https://web.archive.org/web/20221130065156if_/https://www.youtube.com/embed/DZTaiESWl5o](https://web.archive.org/web/20221130065156if_/https://www.youtube.com/embed/DZTaiESWl5o)

// our second talk

## 在 Apache Lucene 中动态生成同义词
的 Word2Vec 模型

14th June, 2:50–3:30 (GMT+2), Kesselhaus

如果您想在 Apache Lucene 中用同义词扩展查询/文档，您需要一个预定义的文件，其中包含具有相同语义的术语列表。找到一种语言的基本同义词列表并不总是容易的，即使你找到了，也不一定与你的上下文领域相匹配。

操作系统文章中的术语“守护进程”不是“魔鬼”的同义词，但它更接近于术语“进程”。

Word2Vec 是一个两层神经网络，它将文本作为输入，并输出字典中每个单词的向量表示。两个意思相近的词用两个彼此接近的向量来标识。

这篇演讲探讨了我们对 Apache Lucene 的贡献，它将这种技术与文本分析管道相结合。
我们将展示如何从 Apache Lucene 索引中动态地自动生成同义词，以及如何将这个新特性与 Apache Solr 一起使用

// slides

[https://web.archive.org/web/20221130065156if_/https://www.slideshare.net/slideshow/embed_code/key/NsE4ncJWGdeRLL?hostedIn=slideshare&page=upload](https://web.archive.org/web/20221130065156if_/https://www.slideshare.net/slideshow/embed_code/key/NsE4ncJWGdeRLL?hostedIn=slideshare&page=upload)

// our speakers![](img/2313dd9fdbaf9f128f257dc8294956f6.png)[](https://web.archive.org/web/20221130065156/https://mobile.twitter.com/dantuzi)*[](https://web.archive.org/web/20221130065156/https://www.linkedin.com/in/daniele-antuzi-6773a282/)* **#### 丹尼尔·安图兹

SOFTWARE ENGINEER @ Sease** **![](img/e462262b74a50542ca59763211b17363.png)[](https://web.archive.org/web/20221130065156/https://twitter.com/ilariapet)*[](https://web.archive.org/web/20221130065156/https://www.linkedin.com/in/ilaria-petreti-422119104/)* **#### [Ilaria Petreti](https://web.archive.org/web/20221130065156/https://sease.io/about/ilaria-petreti)

R&D; SOFTWARE ENGINEER @ Sease**** ****// video

[https://web.archive.org/web/20221130065156if_/https://www.youtube.com/embed/rKYJQhZxQFQ](https://web.archive.org/web/20221130065156if_/https://www.youtube.com/embed/rKYJQhZxQFQ)******