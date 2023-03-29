# SolrCloud 领导人选举失败

> 原文：<https://web.archive.org/web/sease.io/2018/05/solrcloud-leader-election-failing.html>

目前(Solr 7.3.0 ) SolrCloud 是 Apache Solr 的一个可靠而稳定的分布式架构。但是它并不完美，失败时有发生。
这篇 lightning 博客文章将提供一些实用的技巧，当一个集合的一个特定碎片因没有领导者而停止工作，情况陷入僵局时，可以遵循这些技巧。
以下 Solr 版本遇到了以下问题:

*   *   4.10.2
    *   5.4.0

解决问题的步骤可能包括与动物园管理员团队的人工互动[【1】](https://web.archive.org/web/20220930011150/https://lucene.apache.org/solr/guide/7_3/command-line-utilities.html)。
以下步骤摘自 Solr 用户邮件列表[【2】](https://web.archive.org/web/20220930011150/http://lucene.472066.n3.nabble.com/Node-not-recovering-leader-elections-not-occuring-td4287819.html)的一个有趣的线索和该领域的实践经验。特别要感谢 Jeff Wartes 的建议，这些建议在很多情况下对我很有用。

## 问题

*   *   集合中一个分片的所有节点都已启动并运行
    *   碎片没有领袖
    *   所有节点都处于“正在恢复”/“恢复失败”状态
    *   搜索停止，这种情况持续了许多分钟(> 5 分钟)

## 解决办法

发生这种问题的一种可能的解释是，当 Zookeeper clusterstate 的节点本地版本与集中式 Zookeeper 集群状态发生偏离时。
领导者选举失败的一个可能原因是 Zookeeper 故障:例如，您在某段时间内失去了> =50%的集合节点或集合节点之间的连接(这是我直接试验的场景)
这个故障，即使后来得到解决，也会给 Zookeeper 文件系统带来损坏。
某些 SolrCloud 集合可能会保持不一致的状态。

可能需要从 Zookeeper 中手动删除损坏的文件:
让我们从:

***集合/ <集合>/leader _ elect/shard<x>/election*** 一个健康的 SolrCloud 集群呈现的 core_nodeX 与 shard 的副本总数一样多。
您不希望这里出现重复或缺失的节点。
如果你很难获得一个合理的选举，你可以尝试删除编号最低的条目(以及任何编号较低的重复条目)，并尝试再次聚焦选举。随后可能用编号最低的条目重新启动节点。

***集合/ <集合>/leader/shard<x>***
确保该文件夹存在，并且具有作为领导者的预期副本。

***集合/ <集合>/leader _ initiated _ recovery***
该文件夹也可以提供信息，这表示*leader*认为不同步的副本，通常是由于更新请求失败。

完成上述验证后，有几个集合 API 端点可能会有用:

力领袖选举
***/管理/收藏？action = force leader&collection =<collection name>&shard =<shard name>***

强制领袖再平衡
***/管理/收藏？action = rebalance leaders&collection = collection name***

**注意:**重新平衡所有领导者将影响所有碎片

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930011150/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和[elastic search 初学者](https://web.archive.org/web/20220930011150/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930011150/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于 SolrCloud 领袖选举失败的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！