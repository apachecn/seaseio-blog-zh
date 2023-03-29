# Apache Solr 分布式方面

> 原文：<https://web.archive.org/web/sease.io/2018/10/apache-solr-distributed-facets.html>

Apache Solr distributed faceting feature has been introduced back in 2008 with the first versions of Solr (1.3 according to this jira[1]) . Until now, I always assumed it just worked, without diving too much into the details. Nowadays distributed search and faceting are extremely popular, you can find them pretty much everywhere (in the legacy or SolrCloud form alike). **N.B.** Although the mechanics are pretty much the same, Json faceting revisits this approach with some change, so we will now focus on **legacy field faceting**. I think it’s time to get a better understanding of how it works:

## 多个分片请求

When dealing with distributed search and distributed aggregation calculations, you are going to see multiple requests going back and forth across the shards. They have different focus and are meant to retrieve the different bits of information necessary to build the final response. We are going to explore the different rounds of requests, focusing just for the faceting purpose. **N.B.** Some of these requests are also carrying results for the distributed search calculation, this is used to minimise the network traffic. For the sake of this blog let’s simulate a simple sharded index, white space tokenization on field1 and facet.field=**field1**

| 碎片 1 | 碎片 2 |
| *Doc0* *{ "id":"1 "，**" field 1 ":" a b "**}* | *Doc3* *{ "id":"4 "，**" field 1 ":" b c "**}* |
| *Doc1* *{ "id":"2 "，**" field 1 ":" a "**}* | *Doc4* *{ "id":"5 "，**" field 1 ":" b c "**}* |
| *Doc2* *{ "id":"3 "，**" field 1 ":" b c "**}* | *Doc53* *{ "id":"6 "，**" field 1 ":" c "**}* |

**Global Facets** : b(4), c(4), a(2) **Shard 1 Local Facets** : a(2), b(2), c(1) **Shard 2 Local Facets** : c(3), b(2)

## 候选方面字段值的集合

The first round of requests is sent to each shard to identify the candidate top K global facet values. To achieve this target each shard will be requested to respond with its local top K+J facet values and counts. The reason we actually ask for more facets from each shard is to have a better term coverage, to avoid losing relevant facet values and to minimise the refinement requests. How many more we request from each shard is regulated by the “overrequest” facet parameter, a factor that gives more accurate facets at the cost of additional computations[2]. Let’s assume we configure a ***facet.limit=2&facet.overrequest.count=0&facet.overrequest.ratio=1*** to explain when refinement happens and how it works. **Shard 1 Returned Facets** : a(2), b(2) **Shard 2 Returned Facets** : c(3), b(2)

## 收集的计数的全局合并

The facet value counts collected from each shard are merged and the most occurring global top K is calculated. These facet field values are the first candidates to be the final ones. In addition to that, other candidates are extracted from the terms below the top K, based on the shards that didn’t return those values statistics. At this point we have a candidate set of values and we are ready to refine their counts where necessary, asking back this information to the shards that didn’t include that in the first round. This happens including the following specific facet parameter to the following refinement requests: {!terms=$<field>__terms}<field>&<field>__terms=<values> e.g. {!terms=$field1__terms}field1&field1__terms=term1,term2 **N.B.** This request is specifically asking a Solr instance to return back the facet counts just for the terms specified[3] **Top 2 candidates** = b(4), c(3) **Additional candidates** = a(2) The reason that **a(2)** is added to the potential candidates is because Shard 2 didn’t answer with a count for **a,** the potential missing count of 1 could bring **a** to the top K. So it is worth a verification. Shard 1 didn’t return any value for the candidate **c** facet. So the following request is built and sent to it: *facet.field={!terms=$field1__terms}field1&field1__terms=c* Shard 2 didn’t return any value for the candidate **a** facet. So the following request is built and sent to it: *facet.field={!terms=$field1__terms}field1&field1__terms=a*

## 最终计数细化

The refinements counts returned by each shard can be used to finalise the global candidate facet values counts and to identify the final top K to be returned by the distributed request. We are finally done! **Shard 1 Refinements Facets** : c(1) **Shard 2 Refinements Facets** : a(0) **Top K candidates updated** :  **b(4), c(4)**, a(2) GIven a facet.limit=2 the final global facets with correct results returned is : **b(4), c(4)**   [1] [https://issues.apache.org/jira/browse/SOLR-303](https://web.archive.org/web/20220930001324/https://issues.apache.org/jira/browse/SOLR-303) [2] [https://lucene.apache.org/solr/guide/6_6/faceting.html#Faceting-Over-RequestParameters](https://web.archive.org/web/20220930001324/https://lucene.apache.org/solr/guide/6_6/faceting.html#Faceting-Over-RequestParameters) [3] [https://lucene.apache.org/solr/guide/7_5/faceting.html#limiting-facet-with-certain-terms](https://web.archive.org/web/20220930001324/https://lucene.apache.org/solr/guide/7_5/faceting.html#limiting-facet-with-certain-terms)// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930001324/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220930001324/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930001324/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 分布式方面的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！