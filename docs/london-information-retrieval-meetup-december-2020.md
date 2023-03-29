# 伦敦信息检索会议(2020 年 12 月)

> 原文：<https://web.archive.org/web/sease.io/2020/11/london-information-retrieval-meetup-december-2020.html>

![london information retrieval meetup](img/724bf9a3ead4c910f6f7f062bd5bd11c.png)

我们非常高兴地宣布第七届伦敦信息检索聚会，这是一次**免费的**晚间聚会，旨在**对探索和讨论该领域最新趋势感兴趣的信息检索爱好者和专业人士**。

考虑到新冠肺炎的情况和现场直播的可能性，这次我们去了完全远程的**。**

 **晚会将由 1 个技术讲座组成，然后是问答环节，由观众选择问题(将您的问题作为[评论添加到活动](https://web.archive.org/web/20220929233350/https://www.meetup.com/London-Information-Retrieval-Meetup-Group/events/274765296/)！).聚会将以一个网络会议结束。** **![](img/579acfccba303225fdb1c8948a8ea95f.png)// LONDON INFORMATION RETRIEVAL MEETUP

## 程序

在我们的创始人 Alessandro Benedetti 发表简短的欢迎和最新消息讲话后，我们将进行第一场演讲。

// first talk

## 用弹性堆栈丰富邮政地址

大多数时候，我们的客户或用户的邮政地址在我们的信息系统中不是很好地格式化或定义的。举例来说，如果你是一个呼叫中心的员工，想通过客户的地址找到客户，这可能会成为一场噩梦。想象一下，销售服务如何轻松地在地图上标出客户的位置，以及他们可以在哪里开设新店……
让我们举一个简单的例子:

```
{
  "name": "Joe Smith",
  "address": {
    "number": "23",
    "street_name": "r verdiere",
    "city": "rochelle",
    "country": "France"
  }
}
```

或者相反。我有坐标，但我不知道对应的邮政地址是什么:

```
{
  "name": "Joe Smith",
  "location": {
    "lat": 46.15735,
    "lon": -1.1551
  }
}
```

在这个现场编码会议中，我将向您展示如何使用 Elastic stack 解决所有这些问题，重点是 Logstash 和 Elasticsearch。

// speaker![](img/c2d841d940e0ca04906192baad0bf8d8.png)[](https://web.archive.org/web/20220929233350/https://twitter.com/dadoonet)*[](https://web.archive.org/web/20220929233350/https://www.linkedin.com/in/dadoonet/)* **#### 大卫·皮拉托

Evangelist @ Elastic** **David Pilato 是 elastic 和法语口语用户组 creator 的开发人员和布道者。在空闲时间，他喜欢在会议或公司里谈论弹性搜索** **// video

[https://web.archive.org/web/20220929233350if_/https://www.youtube.com/embed/hknFCHf9hr0](https://web.archive.org/web/20220929233350if_/https://www.youtube.com/embed/hknFCHf9hr0)

// Q&A; - question 1

## 自然语言搜索、语言建模和神经搜索

在我们的伦敦信息检索会议上，回答了一些关于自然语言搜索、语言建模(Google Bert、OpenAI GPT-3)、神经搜索和学习排序的问题。

// speaker![](img/9389738dfd041ba28a065dd6c0e3b99a.png)[](https://web.archive.org/web/20220929233350/https://twitter.com/AlexBenedetti)*[](https://web.archive.org/web/20220929233350/https://www.linkedin.com/in/alexbenedetti/)* **#### 亚历山德罗·贝内代蒂

Founder @ Sease** ****阿帕奇 SOLR PMC 成员**
阿帕奇卢森/SOLR 委员

Alessandro 从 2010 年 Apache Solr 1.4 和 edismax query parser 的早期阶段就开始参与设计和开发搜索相关的解决方案。多年来，他一直致力于各种项目，旨在建立能够满足用户信息需求的搜索解决方案，通常将这些解决方案与机器学习和人工智能技术相结合。** **// video

[https://web.archive.org/web/20220929233350if_/https://www.youtube.com/embed/BIILaSb4aRY](https://web.archive.org/web/20220929233350if_/https://www.youtube.com/embed/BIILaSb4aRY)

// Q&A; - question 2

## 学习对图书馆的利弊进行排序

在我们的伦敦信息检索会议上，回答了一些关于自然语言搜索、语言建模(Google Bert、OpenAI GPT-3)、神经搜索和学习排序的问题。

// speakers![](img/6909b1d822883a2a42e7c58809c9ae36.png)[](https://web.archive.org/web/20220929233350/https://www.linkedin.com/in/ilaria-petreti-422119104/) *#### [Ilaria Petreti](https://web.archive.org/web/20220929233350/https://sease.io/ilaria-petreti)

IR/ML ENGINEER @ Sease* *Ilaria 是一名对人工智能世界充满热情的数据科学家。她获得了数据科学硕士学位，坚信大数据和数字化转型的力量。由于在论文工作期间开发了航班延误预测的实际应用，她实施了几种数据挖掘和机器学习技术，并熟悉了编程语言 r。她还参与了一个研究项目，加深了她对集成学习的了解，特别关注超级学习器算法。* *![](img/76b6865f2adaa3cf4172685c0ec290cf.png)[](https://web.archive.org/web/20220929233350/https://www.linkedin.com/in/anna-ruggero-482902153/) *#### 安娜·鲁杰罗

R&D; SOFTWARE ENGINEER @ SEASE* *Anna Ruggero 是一名热衷于信息检索和数据挖掘的软件工程师。她喜欢为问题寻找新的解决方案，建议和测试新的想法，尤其是那些关于将机器学习技术集成到信息检索系统中的想法。Anna 在学习期间接触了搜索引擎，并爱上了这个世界，因此她决定进一步研究这个主题，参加了第 12 届欧洲信息检索暑期学校，并做了关于实体搜索的硕士学位论文。
得益于这条道路，她扩展并提高了自己在 Java 和 Python 语言、信息检索系统、聚类和单词嵌入方面的知识。* *// video

[https://web.archive.org/web/20220929233350if_/https://www.youtube.com/embed/1oQWtv75SJg](https://web.archive.org/web/20220929233350if_/https://www.youtube.com/embed/1oQWtv75SJg)

// slides

[//web.archive.org/web/20220929233350if_/https://www.slideshare.net/slideshow/embed_code/key/L0EN5WmlmcS0b0](//web.archive.org/web/20220929233350if_/https://www.slideshare.net/slideshow/embed_code/key/L0EN5WmlmcS0b0)

**[Interactive Questions and Answers - London Information Retrieval Meetup](//web.archive.org/web/20220929233350/https://www.slideshare.net/SeaseLtd/interactive-questions-and-answers-london-information-retrieval-meetup "Interactive Questions and Answers - London Information Retrieval Meetup")** from **[Sease](https://web.archive.org/web/20220929233350/https://www.slideshare.net/SeaseLtd)**// london information retrieval meetup

## 加入我们的团体

信息检索、机器学习和数据科学领域的研究人员、科学家和其他从业者…加入我们，让我们创建一个充满激情的专业团队！

[Join now](https://web.archive.org/web/20220929233350/https://www.meetup.com/London-Information-Retrieval-Meetup-Group)********