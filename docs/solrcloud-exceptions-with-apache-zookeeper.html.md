# Apache Zookeeper 的 SolrCloud 异常

> 原文：<https://web.archive.org/web/sease.io/2018/05/solrcloud-exceptions-with-apache-zookeeper.html>

目前(Solr 7.3.1 ) SolrCloud 是 Apache Solr 的一个可靠而稳定的分布式架构。但是它并不完美，失败时有发生。

Apache Zookeeper[【1】](https://web.archive.org/web/20230226232125/https://zookeeper.apache.org/)是负责管理整个 SolrCloud 集群通信的系统。
它包含共享集合配置，并具有集群状态视图。
它是集群大脑的一部分，是维持集群健康运转的守护者。

它能够回答以下问题:

谁是这个碎片和系列的领导者？
此节点是否关闭？
这个节点正在恢复吗？

Solr 节点与 Zookeeper 通信，以了解在运行 SolrCloud 操作时应该联系谁。

这篇关于 lightening 的博文将提供一些实用的技巧，供您的客户端应用程序在处理 SolrCloud 和 Apache Zookeeper 时遇到一些经典异常时参考。
特别感谢 Apache Solr 用户邮件列表贡献者和 Apache Solr 社区，这篇文章汇集了来自他们以及官方代码和文档的建议。

## org . Apache . Solr . common . Solr exception:
无法从 ZK 加载集合:
<集合名称>

如果您在这里登陆时只出现了那个异常，我假设有一个缺失:
“由:org . Apache . zookeeper . keeper Exception $ SessionExpiredException:keeper error code = Session expired for/collections/<collections name>/state . JSON”导致？

Solr 的**zkclientimeout**用于设置 ZooKeeper 的 sessionTimeout，这是一个会话到期时所超出的。
当这种异常发生时，这意味着 Solr-Zookeeper 通信中出现了非常严重的问题，30 秒(当前默认的[【2】](https://web.archive.org/web/20230226232125/https://issues.apache.org/jira/browse/SOLR-5565))是应用程序尝试通信的一段非常长的时间。

**推荐**:照顾好身边不同的暂停时间，不要让它们太小！
即，为 zkClientTimeout 指定一个值> = 30 秒**。**

**maxSessionTimeout**(Zookeeper)
3 . 3 . 0 中新增:服务器允许客户端协商的最大会话超时时间(毫秒)。默认为 tickTime 的 20 倍。

**zkclienttime out**(Solr)
控制你的客户端超时。

检查超时后，让我们探索一些可能的根本原因。
会话过期可能由以下原因引起:

1.Solr node/Zookeeper 上的垃圾收集——当堆太小或太大时，会发生极端的垃圾收集暂停。磁盘上的 IO 变慢。
3。网络延迟

**建议**

1.  设置一个 JVM 分析器来密切监视您的 Solr 和 Zookeeper 节点，特别注意垃圾收集周期和一般的内存使用情况:您不希望 Zk 交换太多！(GC viewer[【3】](https://web.archive.org/web/20230226232125/https://github.com/chewiebug/GCViewer)可能是一个很好的工具)
2.  验证 Zookeeper 节点可以快速写入磁盘:Zookeeper 需要快速写入，最好分配一个单独的磁盘。
3.  监控您的网络，确保 solr 节点可以有效地与 Zookeeper 节点通信

如果这些建议没有解决您的问题，您可能遇到了 Solr bug。
其中一个是[【4】](https://web.archive.org/web/20230226232125/https://issues.apache.org/jira/browse/SOLR-8868)，不幸的是还没有修复。

## org . Apache . Solr . client . solrj . SolrServersexception:
没有可用的活动 solrserver 来处理这个问题

来自官方 JavaDoc:

***org/Apache/Solr/client/solrj/impl/lbhttpsolrclient . Java:369***
"尝试从 Req 中提供的列表中查询实时服务器。将跳过死池中的服务器。
*如果请求因 IOException 而失败，服务器将被移至死池一段时间
*或直到该服务器上的测试请求成功。
*
*按照给定的顺序查询服务器(跳过当前在死池中的服务器除外)。
*如果所提供的列表中没有剩余的活动服务器需要尝试，将会尝试许多之前跳过的失效服务器。
* req . getnumdeadserverstory()控制将尝试多少个失效的服务器。
*
*如果没有找到活动服务器，将引发 SolrServerException。

异常发生时集群的状态是什么？根据 Zookeeper 的知识，有任何 Solr 服务器启动并运行吗？

建议在异常发生时检查 clusterstate.json。
从 Solr admin UI 中，您可以打开 Cloud- >树并验证哪些节点已启动并正在运行。

这可能与节点故障有很大关系(这可能与任何可能的原因有关，包括 GC)
我见过由特定查询引起的情况，真正的异常被“没有活动的 SolrServers…”客户端异常所隐藏。Solr 日志应该有助于识别内部 Solr 问题，JVM 监控可以丢弃任何内存/gc 问题。
有些人通过通配符查询看到了这一点(当每个分片都报告“太多扩展…”类型错误，但是客户端响应中的异常是“没有活动的 SolrServers…”)。

## org . Apache . Solr . common . sol rexception:
找不到健康的节点来处理请求

与“没有实时 Solr 服务器”的考虑差不多。当负载平衡器 SolrJ 端无法从集群中检索到活动节点时，会发生这种情况(基于 Zookeeper 状态)。
这发生在前一个异常之前，所以请求甚至没有到达 LoadBalancinghttpSolrClient。

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20230226232125/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20230226232125/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20230226232125/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Zookeeper 的 SolrCloud 异常的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！