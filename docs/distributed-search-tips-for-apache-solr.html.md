# Apache Solr 的分布式搜索技巧

> 原文：<https://web.archive.org/web/sease.io/2017/11/distributed-search-tips-for-apache-solr.html>

分布式搜索是 Apache Solr 可伸缩性的基础:

可以跨同一集合的不同 Apache Solr 节点进行分布式搜索(在传统的[【1】](https://web.archive.org/web/20220929234349/https://lucene.apache.org/solr/guide/6_6/distributed-search-with-index-sharding.html)或 SolrCloud[【2】](https://web.archive.org/web/20220929234349/https://lucene.apache.org/solr/guide/6_6/solrcloud.html)架构中)，但也可以跨 Solr cloud 集群中的不同集合进行分布式搜索。
当您将不同的系统(本应是独立的)放在适当的位置，并且您后来意识到聚合结果可能是一个额外的有用用例时，聚合来自不同集合的结果可能是有用的。
这篇博客将关注在运行分布式搜索时可能发生的一些棘手情况(关于配置或细节，您可以参考 Solr wiki)。

## 综合资料的文件（intergrated Data File）

逆文档频率影响得分。
这意味着**与来自较小集合的类似文档相比，来自较大集合的文档可以从 IDF 获得提升**。
这是因为 maxDoc 计数被考虑为语料库大小，因此即使一个术语具有相同的文档频率，IDF 也会受到集合大小的强烈影响。
分布式 IDF[【3】](https://web.archive.org/web/20220929234349/https://lucene.apache.org/solr/guide/6_6/distributed-requests.html#DistributedRequests-ConfiguringstatsCache_DistributedIDF_)部分解决了问题:

当在同一个集合的不同片段之间分配搜索时，它工作得相当好。
但是**使用 ExactStatCache 并在同一个 SolrCloud 集群中交替使用单集合分布和多集合分布会产生一些缓存冲突。**

具体来说，如果我们首先执行集合间查询，缓存的全局统计信息将是集合间全局统计信息，因此，如果我们随后执行单个集合分布式搜索，预览全局统计信息将保持缓存(反之亦然)。

## 调试评分

真实分数和调试分数与分布式 IDF 不一致，这意味着调试查询将不会显示正确的分布式 IDF 和用于分布式搜索的正确分数计算[【4】](https://web.archive.org/web/20220929234349/https://issues.apache.org/jira/browse/SOLR-7759)。

## 相关性调整

Lucene/Solr 分数不是概率性的或标准化的。对于同一个集合，我们可以通过不同的查询获得完全不同的评分标准。
当我们**调整我们的相关性，增加乘法或加法提升**时，情况变得更加复杂。
不同的集合可能意味着完全不同的提升逻辑，这可能导致一个集合的分数与另一个集合相比处于完全不同的等级。
在调整不同集合的搜索相关性时，我们需要格外小心，并尽可能以最兼容的方式配置分布式请求处理程序。

## 请求处理程序

使用分布式搜索时，仔细指定要使用的请求处理程序非常重要。
请求将命中一个节点中的一个集合，然后在分布时，相同的请求处理程序将被跨其他节点的其他集合调用。
如有必要，可以配置聚合器请求处理器和本地请求处理器(如果我们希望使用本地参数对每个集合使用不同的评分公式，这可能会很有用) :

#### 聚合器请求处理程序

它在接收请求的第一个节点上执行。
它将分发请求，然后汇总结果。
它必须描述搜索中所有集合共有的参数。
是主请求中指定的。
例如
*[http://localhost:8983/Solr/collection 1/](https://web.archive.org/web/20220929234349/http://localhost:8983/solr/collection1/)**选择**？*

#### 本地请求处理程序

它是通过参数指定的: **shards.qt=** 在第一个接收到分布式查询的节点上执行。
这可用于在每个集合的基础上使用特定字段或参数。
本地请求处理器可以使用感兴趣的集合本地的字段和搜索组件。

例如
[http://localhost:8983/Solr/collection 1/select？q=*:* &集合=集合 1，集合 2](https://web.archive.org/web/20220929234349/http://localhost:8983/solr/collection1/select?q=*:*&collection=collection1,collection2) **&碎片. Qt =局部选择**

**N.B.** 如果您想要定义本地查询解析器规则，例如影响分数的本地 edismax 配置，使用本地请求处理程序可能会很有用。

## 唯一键

不同集合的唯一键字段必须相同**。** 此外，该值在不同的集合中应该是唯一的，以保证正确的行为。如果我们不遵守这条规则，Solr 将无法聚集结果并引发一个异常。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929234349/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929234349/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929234349/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 分布式搜索技巧的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！