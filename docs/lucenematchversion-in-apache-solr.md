# Apache Solr 中的 luceneMatchVersion 参数

> 原文：<https://web.archive.org/web/sease.io/2021/04/lucenematchversion-in-apache-solr.html>

Apache Solr solrconfig.xml 中的 luceneMatchVersion 参数指定了一个参考 Apache Lucene 版本，用于影响一些内部组件。Apache Solr 使用 Apache Lucene 作为内部库，Apache Solr 版本的二进制文件与特定版本的 Lucene 库相耦合。

例如
Apache Solr 8.8.1 版本使用 Apache Lucene 8.8.1 库
你可以在:*…/Solr-8 . 8 . 1/server/Solr-WEB app/WEB app/we b-INF/lib*ls | grep Lucene 下找到这样的库

*   *   Lucene-backward-codecs-8 . 8 . 1 . jar
    *   Lucene-分类-8.8.1.jar
    *   lucene-codecs-8.8.1.jar
    *   lucene 核心 8.8.1.jar
    *   …

**注意**在 2021 年 2 月 17 日 Apache Lucene 和 Solr 拆分后，未来版本可能不一致，即 Apache Solr X 可能使用 Apache Lucene Y

【T8

那么假设一个 Apache Solr 版本与一个精确的 Apache Lucene 版本相结合，那么 luceneMatchVersion 配置的意义和用法是什么呢？

## 支持的值

*   *   特定版本，例如 8.8.1(主要版本、次要版本、错误修复)
    *   LATEST，LUCENE_CURRENT ->两者都映射到 Apache Solr 二进制文件中包含的 Apache Lucene 版本

这个类中列出了与 Lucene 版本相关的支持版本列表:*org . Apache . Lucene . util . version*

**注意**给定一个使用特定 Apache Lucene 版本的 Apache Solr 版本，luceneMatchVersion 的支持值将返回到主版本号版本-1
例如
Apache Solr **8.8.1** 使用 Apache Lucene **8.8.1** 并支持返回到**7.0**
Apache Solr**7 的 luceneMatchVersion。**

如果您设置了不支持的 luceneMatchVersion，您会在日志中发现警告:
例如
**8 . 8 . 1**with<luceneMatchVersion>**6.6.5**</luceneMatchVersion>6 . 6 . 5<7 . 0 . 0(8-1)
*…正在使用不推荐的 6 . 6 . 5 仿真。您应该在某个时候声明并重新索引到至少 7.0，因为 6.x 仿真已被弃用，并将在 8.0 中删除*

## 不改变索引数据结构

一个常见的误解是，在 Apache Solr 版本 X 中设置 <lucenematchversion>Y</lucenematchversion> ，将使 Solr 使用 Y Lucene 索引格式(使用 Y 编解码器和 Y 数据结构)。
**不是这样的**，Apache Solr 版总是会构建一个 Apache Lucene 索引，再加上 Solr 中包含的内部库版本。
例如
Solr 8.8.1 使用 Lucene 8.8.1 总是独立于 luceneMatchVersion 构建 Lucene 8.8.1 索引。
luceneMatchVersion 是 Solr 代码中各种条件检查的一部分，这可能会改变一些组件行为，让我们来详细了解一下。

## 版本升级-文本分析

luceneMatchVersion 参数主要是一个工具，用于在升级过程中确保一致的索引和查询行为。
新版本可能会为某些字段类型的文本分析链引入不同的行为(标记器、标记过滤器等..).令牌化器中的一个 bug 可以被修复，或者只是令牌过滤器的工作方式可以被改变。
当您将 Apache Solr 实例升级到 X+1 版本时，如果您想要保持与旧的 Lucene 版本 X 相同的逻辑，为了与您正在使用的文本分析链保持一致，在 luceneMatchVersion 中设置这样的版本 X 是一个好主意。
将 Solr 实例从版本 **X** 升级到 **X+1** 时，最好使用**<luceneMatchVersion>X</luceneMatchVersion>**部署新版本。通过这种方式，您可以保持与旧索引的一致性，并继续对新文档进行索引，从而最大限度地减少意外(至少为了向后兼容),直到您有能力重新索引为止。
你应该尽快将 luceneMatchVersion 升级到 **X+1** 并运行**重新索引**。

这是因为新的 Solr 可以读取某个旧的索引版本，所以现有的索引段将保持原来的
格式，而新的索引段将以新的格式写入。
如果由于合并策略合并了任何现有的段，那么新的更大的段将采用新的格式。

例如
如果一个索引以 6.x 开始，然后在 7.x 中运行一段时间，但仍有 6.x 段(未合并)，那么该索引在 8.0 中将不起作用(独立于 luceneMatchVersion)

## 版本升级-为什么你不应该使用“最新的”

如果您设置了 <lucenematchversion>LATEST</lucenematchversion> ，那么您将无法控制与 Apache Solr 集合相关联的确切的 luceneMatchVersion(它将是 Solr 二进制代码的相同版本)。
因此，如果你进行升级，你可能会在升级后的文本分析和其他组件中出现无法预料的变化。如果精确的向后兼容性很重要，你应该总是指定一个精确的版本。

## 评分算法(Lucene/Solr 中的相似性)

Apache Lucene 中的相似性算法实现了在查询时进行排名时为搜索结果分配分数的逻辑。
实现了各种相似度，你可以在这里找到:
Lucene/Lucene/core/src/Java/org/Apache/Lucene/search/Similarity
目前在 Apache Solr 经典相似度是 TF-IDF([https://en.wikipedia.org/wiki/Tf–idf](https://web.archive.org/web/20220930001141/https://en.wikipedia.org/wiki/Tf%E2%80%93idf))schema Similarity 是 BM25([https://en.wikipedia.org/wiki/Okapi_BM25](https://web.archive.org/web/20220930001141/https://en.wikipedia.org/wiki/Okapi_BM25))。
从 6.0 开始，BM25 被引入作为 Apache Lucene/Solr 的缺省值。

T13

在 org . Apache . solr . search . Similarity . schemasimilarityfactory # get Similarity 中，luceneMatchVersion 规定了默认情况下使用哪个相似性算法:
**N.B.** 这个代码片段来自 Solr 6，在当前的 Solr 实现中，条件检查中涉及 BM25Similarity 和 LegacyBM25Similarity，但概念是相同的。

```
defaultSim = this.core.getSolrConfig().luceneMatchVersion.onOrAfter(Version.LUCENE_6_0_0)
           ? new BM25Similarity()
           : new ClassicSimilarity();
```

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930001141/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220930001141/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)吗？我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930001141/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 中 LuceneMatchVersion 参数的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！