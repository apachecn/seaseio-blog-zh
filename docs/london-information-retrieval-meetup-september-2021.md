# 伦敦信息检索会议[2021 年 9 月]

> 原文：<https://web.archive.org/web/sease.io/2021/08/london-information-retrieval-meetup-september-2021.html>

![](img/85096f0d268333cece5f44126922ef81.png)

我们非常高兴地宣布第十届伦敦信息检索聚会，这是一次**免费的**晚间聚会，旨在**对探索和讨论该领域最新趋势感兴趣的信息检索爱好者和专业人士**。

这一次 meetup 将是一个混合活动**和在线活动**。现场活动将是**[Desires 2021](https://web.archive.org/web/20221223104505/http://desires.dei.unipd.it/)—****实验搜索&信息检索系统**会议的卫星活动。

### 现场会议

**地点:**帕多瓦大学信息工程系-帕多瓦(意大利)|奥拉马格纳莱普斯基



**日期**:2021 年 9 月 15 日|下午 3:00-5:00(GMT+1)

### 在线聚会

**需要注册**

**日期**:2021 年 9 月 15 日|下午 3:00-5:00(GMT+1)

本次活动将由两个技术讲座组成，每个讲座后会有一个问答环节。

// LONDON INFORMATION RETRIEVAL MEETUP

## 程序

在我们的创始人 Alessandro Benedetti[](https://web.archive.org/web/20221223104505/https://sease.io/alessandro-benedetti)**简短的欢迎和最新消息演讲后，我们将进行第一场演讲。**

**// first talk

## 评级评估企业:下一代免费搜索质量评估工具

每个信息检索从业者都在努力评估搜索引擎如何满足用户的信息需求。提高搜索系统的正确性和有效性需要一套有助于衡量这些方面的工具。**回到 2018 年，评级评估师(RRE)来拯救我们。**

RRE 是一个开源的搜索质量评估工具，可以用来生成一组关于系统质量的报告，一次又一次的迭代，并且可以集成到一个持续集成基础设施中，以在每次发布后监控质量度量。

尽管许多方面仍然存在问题:

–如何直接评估一个与 Apache Solr 或 Elasticsearch 通信的中间层 search-API？
–如何轻松生成显式和隐式评级，而无需花费数小时在繁琐的 json 文件上？
–如何更好的发掘评价结果？有漂亮的部件和有趣的见解？

评级评估企业解决了这些问题和更多。

// speakers![](img/71b6aac32e38e4cd919363a283ba286d.png)[](https://web.archive.org/web/20221223104505/https://twitter.com/AlexBenedetti)*[](https://web.archive.org/web/20221223104505/https://www.linkedin.com/in/alexbenedetti/)* **#### 亚历山德罗·贝内代蒂

Founder @ Sease**  ****阿帕奇 SOLR PMC 成员**
阿帕奇卢森/SOLR 委员

Alessandro 从 2010 年 Apache Solr 1.4 和 edismax query parser 的早期阶段就开始参与设计和开发搜索相关的解决方案。多年来，他一直致力于各种项目，旨在建立能够满足用户信息需求的搜索解决方案，通常将这些解决方案与机器学习和人工智能技术相结合。** **![](img/6474b1e0b486dbd520d68f87f079f485.png)[](https://web.archive.org/web/20221223104505/https://twitter.com/agazzarini)*[](https://web.archive.org/web/20221223104505/https://www.linkedin.com/in/andreagazzarini/)* **#### 安德烈娅·加扎里尼

Co-Founder @ Sease**  ****RRE 创造者**

Andrea Gazzarini 是一名好奇的软件工程师，主要研究 Java 语言和搜索技术。他在各种软件工程领域拥有超过 15 年的经验，他在搜索世界的冒险始于 2010 年，当时他遇到了 Apache Solr 以及后来的 Elasticsearch。** **// slides

[//web.archive.org/web/20221223104505if_/https://www.slideshare.net/slideshow/embed_code/key/glOaXCgWFKLqiw](//web.archive.org/web/20221223104505if_/https://www.slideshare.net/slideshow/embed_code/key/glOaXCgWFKLqiw)

**[Rated Ranking Evaluator Enterprise: the next generation of free Search Quality Evaluation Tools](//web.archive.org/web/20221223104505/https://www.slideshare.net/SeaseLtd/rated-ranking-evaluator-enterprise-the-next-generation-of-free-search-quality-evaluation-tools "Rated Ranking Evaluator Enterprise: the next generation of free Search Quality Evaluation Tools")** from **[Sease](https://web.archive.org/web/20221223104505/https://www.slideshare.net/SeaseLtd)**// video

[https://web.archive.org/web/20221223104505if_/https://www.youtube.com/embed/AaR7ZPOBuEs](https://web.archive.org/web/20221223104505if_/https://www.youtube.com/embed/AaR7ZPOBuEs)

// second talk

## 我错过了什么吗？系统综述中基于意图感知的查询性能预测

近年来，信息检索中查询重构的研究引起了广泛的兴趣。事实上，当选择了信息需求的“正确”公式或者当多个公式的结果融合在一起时，系统的性能可以大大提高。

该研究领域的主要挑战之一是能够在可能的变化中预测性能最佳的查询。查询性能预测的一个用例是文献综述的系统汇编。事实上，系统综述是一种科学调查，它使用策略包括对所有潜在相关文章的全面搜索。由于编写系统综述的时间和资源是有限的，因此需要对搜索进行限制:例如，人们可能希望估计搜索的范围应该有多远(即文献中可能存在的所有可能的病例/文献)，以便在资源完成之前停止

在本讲座中，我们分析了用于估计搜索会话期间每个查询重构的丢失信息量的意图感知指标的优点和缺点。

// speaker![](img/0f52c57418078175b4d79186c33ba55f.png)[](https://web.archive.org/web/20221223104505/https://twitter.com/airamoigroig)*[](https://web.archive.org/web/20221223104505/https://it.linkedin.com/in/giorgio-maria-di-nunzio)* **#### [乔治·玛利亚·迪·农齐奥](https://web.archive.org/web/20221223104505/http://www.dei.unipd.it/~dinunzio/MyAcademicPage/Welcome.html)

ASSOCIATED PROFESSOR AT UNIPD****Giorgio Maria Di Nunzio is Associate Professor at the Department of Information Engineering of the University of Padua, Italy. His current research interests include the design of interactive machine learning models for the retrieval of medical and legal documents, and the evaluation of Technology-Assisted Review systems based on Continuous Active Learning. In particular, he is interested in the study of the prediction of the costs of high recall systems related to query reformulation and rank fusion.****// video

[https://web.archive.org/web/20221223104505if_/https://www.youtube.com/embed/19G6pS9ZuMg](https://web.archive.org/web/20221223104505if_/https://www.youtube.com/embed/19G6pS9ZuMg)

// london information retrieval meetup

## 加入我们的团体

信息检索、机器学习和数据科学领域的研究人员、科学家和其他从业者…加入我们，让我们创建一个充满激情的专业团队！

[Join now](https://web.archive.org/web/20221223104505/https://www.meetup.com/London-Information-Retrieval-Meetup-Group)********