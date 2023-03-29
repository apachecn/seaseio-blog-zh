# RRE 企业查询发现

> 原文：<https://web.archive.org/web/sease.io/2022/08/query-discovery-in-rre-enterprise.html>

## “入侵者”层的作用

开源版本的 RRE 之所以能够非常快速且立即投入使用，其中一个原因是 T2 与目标搜索引擎的直接沟通。

这意味着 IDE 中的搜索工程师可以使用 RRE 来绑定和测试一组**评级**，直接指向 **Solr** 或 **Elasticsearch** 实例。

虽然这对于具体地**提高被测系统的搜索质量**无疑是实用的，但是它引入了一个**强妥协**:在系统中定义的、将被执行来创建搜索质量度量的查询不是“**用户查询**”:它们需要使用本地搜索引擎语言来声明。以下是一个 Elasticsearch 查询的示例:

```
{
  "query": {
    "match": {
      "name": {
        "query": "$query",
        "minimum_should_match": "3<-75% 9<-85%"
      }
    }
  }
}

```

同样，这与目标搜索引擎建立了强大而直接的联系，但是……根据我们的经验，这并没有反映出广泛使用/经过验证的三层架构，该架构将系统责任分配给:

*   **客户端应用**:通常是前端层(如 AngularJS 或 ReactJS)
*   **API 层**:中间组件(在微服务架构的情况下实际上是一组组件)，负责通过协调和编排内部子系统(例如 RDBMS、搜索引擎、NoSQL 存储)来隐藏、抽象和实现系统逻辑。
*   **一个“数据源”层**，由一个或多个存储子系统组成。它们中的每一个都管理数据，在某些情况下甚至是相同的数据，用于不同的目的。

**Martin Fowler**, in his famous book [“Patterns of Enterprise Application Architecture”](https://web.archive.org/web/20220929232553/https://martinfowler.com/books/eaa.html) describes that layered architecture as composed of the **Presentation**, the **Domain**, and the **Data Source** layers.

回到我们的搜索质量上下文，这意味着在通常的架构中，一个“入侵者”(API 层)**在**用户查询**和相应的**搜索引擎查询**之间充当中介**。

![](img/a3f354671213ae580dd5c6ceecae929a.png)

API 层实现系统逻辑。这意味着从一个请求开始(在这种情况下，让我们简化一下，称之为搜索 API 请求),有一个业务工作流，它触发涉及几个组件的几个动作。

对于搜索引擎，这意味着 API 层构建并执行特定于搜索引擎的查询，例如考虑 ACL、权限过滤器、增强逻辑等等。

如果我们想实际测量这样一个系统的搜索质量，抛弃 API 层所扮演的角色是正确的吗？肯定不是。不幸的是，这正是开源版本的 RRE 所做的。

理想情况下，我希望能够考虑整个系统，包括 API 层，作为度量的对象。

## RRE 企业:查询发现

RRE 企业通过实现查询发现机制来填补上述空白。它是如何工作的？不涉及技术细节，基本思想非常简单:

*   在评估过程中不要考虑表示层
*   将**查询**实体拆分成两个相关的请求:**搜索 API 请求**和**搜索引擎请求**
*   **向搜索 API 层触发**一个**搜索 API 请求**
*   **捕获**对应的**搜索引擎请求**(在搜索引擎端)
*   **存储**他们之间的**关联**

At the end, a **rating definition** will therefore include all the relevant pieces that contributed to a given **query execution**, including the **Search API request** and the corresponding the **Search Engine requests****RREE** implements the **query** **discovery** described above both in **Apache Solr** and **Elasticsearch**.The **only assumption required** for a **successful discovery** is to have, during that process, **an exclusive access** to the target search engine.As you can imagine, if there is some other process that is using the search engine, it would be very hard in the correlation phase to distinguish between queries executed as consequence of RREE discovery and other applications.    

## 概述

**RREE 查询发现**是**评估基础设施**的重要组成部分:它允许将**搜索应用作为一个整体**来考虑，因此**针对的是**一个严格接近真实生产环境的被评估系统。

它是通过在评估过程中包含 **业务****系统** 逻辑由 **中间 API 层** 来实现的，它们是 **搜索应用** 的关键部分。T52

目的是最大化评估流程输出的“可信度”。

// BEGIN YOUR JOURNEY INTO THE SEARCH QUALITY EVALUATION

## 评级评估企业

[DISCOVER MORE AND DOWNLOAD](https://web.archive.org/web/20220929232553/https://sease.io/rated-ranking-evaluator-enterprise)// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Drop constant features:一个现实世界的学习排名场景的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！