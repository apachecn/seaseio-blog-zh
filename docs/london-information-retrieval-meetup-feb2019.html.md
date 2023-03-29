# 伦敦信息检索会议[2019 年 2 月]

> 原文：<https://web.archive.org/web/sease.io/2019/02/london-information-retrieval-meetup-feb2019.html>

![london information retrieval meetup](img/a765fb7f284bce6442b0cff0d9cf0176.png)

伦敦信息检索会议即将召开(19/02/2019 ),我们很高兴添加更多关于演讲者和演讲的详细信息！

T2T4

// LONDON INFORMATION RETRIEVAL MEETUP

## 程序

在我们的创始人 Alessandro Benedetti 发表简短的欢迎和最新消息演讲后，我们将进行第一场演讲。

// first talk

## 学习排序:为恐龙解释

互联网搜索的发展由来已久，那时你必须以正确的方式将你的查询串起来，才能得到你想要的结果。搜索必须是智能和自然的，人们希望它能够“正常工作”,阅读他们的想法。

另一方面，任何在搜索引擎幕后工作过的人都清楚地知道在正确的时间得到正确的结果有多难。在你的用户可以在首页找到他最喜欢的两条腿的小臂恐龙之前，花了无数个小时来调整增强功能。

当你的数据不断发展、更新时，你的搜索引擎也是如此。因此，搜索团队一直在追求改进和提高其搜索结果的排名和相关性。但是，聪明地工作并不等同于努力工作。我们可以采用许多技术来帮助我们动态地改进和自动化这一过程。其中一个技巧就是学习排名。

大约 20 年前，学术界首次提出学习排名，几乎所有商业网络搜索引擎都以某种形式利用它。在彭博，我们决定是时候让开源搜索引擎支持学习排名了，所以我们花了一年多的时间来设计和实现它。我们努力的结果已经被 Solr 社区接受，我们的学习排名插件现在在 Apache Solr 中可用。

这个演讲将作为 Solr 中 LTR(学习到排名)模块的介绍。不需要关于学习排名的先验知识，但是与会者应该知道 Python、Solr 和机器学习技术的基础。我们将逐步完成在 Solr 中发布机器学习排名模型的过程，包括:

*   如何根据您的需求设计功能和构建训练数据集
*   如何使用 scikit-learn 等流行的 Python ML(机器学习)库来训练排名模型
*   你如何在 Solr 中使用上面学到的排名模型

准备好一个互动的会议，我们学习排名！

// speaker![](img/699d82f1c3b03073486ff5e7c95a24db.png)[](https://web.archive.org/web/20220929233020/https://twitter.com/_sambhavkothari)*[](https://web.archive.org/web/20220929233020/https://linkedin.com/in/sambhav-kothari)* **#### 桑巴夫·科塔里

SOFTWARE ENGINEER @ BLOOMBERG** **Sambhav 是彭博的一名软件工程师，在新闻搜索体验团队工作** **// second talk

## 利用动态规划和长跳转改进 top-k 检索算法

现代搜索引擎必须跟上用户提交的文档和查询数量的巨大增长。要处理的问题之一是为给定的查询找到最佳的 k 个相关文档。这种操作必须很快，只有通过使用专门的技术才有可能。Block max wand 是解决该问题的最著名的算法之一，其排名没有任何有效性下降。
在简单介绍之后，在本次演讲中，我将展示“使用可变大小块的更快 BlockMax WAND”(SIGIR 2017)中介绍的一种策略，该策略应用于 block max WAND 数据，使算法执行速度提高了近 2 倍。
然后，将提出另一种优化的 BlockMaxWand 算法(“更快的 BlockMax WAND 与更长的跳跃”，ECIR 2019)，用于减少短查询的时间执行。

// speaker![](img/f0f9e5de3eb499a17e6eeca3f3fb01b4.png)[](https://web.archive.org/web/20220929233020/https://www.linkedin.com/in/elia-porciani-89730866/) *#### [埃利亚·波恰尼](https://web.archive.org/web/20220929233020/https://sease.io/elia-porciani)

R&D; SOFTWARE ENGINEER @ SEASE* *Elia 是一名软件工程师，热衷于与搜索引擎和效率相关的算法和数据结构。
他目前参与了 CNR(意大利国家研究委员会)的许多研究项目，并且出于个人目的。
在加入 Sease 之前，他在 Intecs 和 List 工作，通过处理嵌入式和联网等低级编程问题，直到高级交易算法，他可以体验不同领域和不同水平的计算机科学。他的毕业论文是关于搜索引擎上的数据压缩和查询性能。
他是信息检索研究界的活跃分子，参加过 SIGIR 和 ECIR 等国际会议。* *// slides

[//web.archive.org/web/20220929233020if_/https://www.slideshare.net/slideshow/embed_code/key/zUCwdsc47176v0](//web.archive.org/web/20220929233020if_/https://www.slideshare.net/slideshow/embed_code/key/zUCwdsc47176v0)

**[Improving Top-K Retrieval Algorithms Using Dynamic Programming and Longer Skipping](//web.archive.org/web/20220929233020/https://www.slideshare.net/SeaseLtd/improving-topk-retrieval-algorithms-using-dynamic-programming-and-longer-skipping "Improving Top-K Retrieval Algorithms Using Dynamic Programming and Longer Skipping")** from **[Sease](https://web.archive.org/web/20220929233020/https://www.slideshare.net/SeaseLtd)**// third talk

## 音乐信息检索导论

音乐信息检索是关于从音乐实体中检索信息。
这个高层次的定义涉及到一个复杂的学科，有许多现实世界的应用。
作为一名前贝司手，Andrea 将描述音乐信息检索的高级概述，并从音乐家的角度分析该主题带来的一系列挑战。
我们将介绍音乐语言的基本概念，然后通过不同种类的音乐表示，我们将最终描述一些在处理音乐实体时使用的有用的低级特征。

// speaker![](img/5b722011ae59fd7bd3c5cd6943dbb1e4.png)[](https://web.archive.org/web/20220929233020/https://twitter.com/agazzarini)*[](https://web.archive.org/web/20220929233020/https://www.linkedin.com/in/andreagazzarini/)* **#### 安德烈娅·加扎里尼

Co-Founder @ Sease**  ****RRE 创造者**

Andrea Gazzarini 是一名好奇的软件工程师，主要研究 Java 语言和搜索技术。他在各种软件工程领域拥有超过 15 年的经验，他在搜索世界的冒险始于 2010 年，当时他遇到了 Apache Solr 以及后来的 Elasticsearch。** **// london information retrieval meetup

## 加入我们的团体

信息检索、机器学习和数据科学领域的研究人员、科学家和其他从业者…加入我们，让我们创建一个充满激情的专业团队！

[Join now](https://web.archive.org/web/20220929233020/https://www.meetup.com/London-Information-Retrieval-Meetup-Group)*****