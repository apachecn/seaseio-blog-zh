# Solr 文件分类-第 1 部分-索引时间

> 原文：<https://web.archive.org/web/sease.io/2015/07/solr-document-classification-part-1-indexing-time.html>

## 介绍

这篇博文是关于 Solr 分类模块和 Lucene 分类在索引时的集成方式。

In the [previous blog](https://web.archive.org/web/20220930000700/https://sease.io/2015/07/lucene-document-classification.html) we have explored the world of Lucene Classification and the extension to use it for Document Classification.It comes natural to integrate Solr with the Classification module and allow Solr users to easily manage the Classification out of the box.

###### **注意:**Solr 6.1支持此功能

## Solr 文件分类

Taking inspiration from the work of a dear friend [[2]](https://web.archive.org/web/20220930000700/https://www.slideshare.net/teofili/text-categorization-with-lucene-and-solr), integrating the classification in Solr can happen 2 sides :

*   索引时间–通过更新请求处理器
*   查询时间——通过请求处理程序(类似于更像这样的)

In this first article we are going to explore the Indexing time integration :The Classification Update Request Processor.

## 分类更新请求处理器

First of all let’s describe some basic concepts :An **Update Request Processor Chain**, associated to an Update handler, is a pipeline of Update processors, that will be executed in sequence.
It takes in input the added Document (to be indexed) and return the document after it has been processed by all the processors in the chain in sequence.Finally the document is indexed.An **Update Request Processor** is the unit of processing of a chain, it takes in input a Document and operates some processing before it is passed to the following processor in the chain if any.The main reason for the Update processor is to add intermediate processing steps that can enrich, modify and possibly filter documents , before they are indexed.
It is important because the processor has a view of the entire Document, so it can operate on all the fields the Document is composed.For further details, follow the official documentation [[3]](https://web.archive.org/web/20220930000700/https://solr.apache.org/guide/6_6/update-request-processors.html).

## 描述

The Classification Update Request Processor is a simple processor that will **automatically classify** a document ( the classification will be based on the latest index available) adding a new field  containing the class, before the document is indexed.After an initial valuable index has been built with human assigned labels to the documents, thanks to this Update Request Processor will be possible to ingest documents with automatically assigned classes.The processing steps are quite simple :When a document to be indexed enters the Update Processor Chain, and arrives to the Classification step, this sequence of operations will be executed :

*   **最新索引阅读器**从最新打开的搜索器中检索
*   用 solrconfig.xml 中的配置参数实例化一个 **Lucene 文档分类器**
*   考虑到来自输入文档的所有相关字段，分类器分配一个**类**
*   一个**新字段被添加**到原始文档，带有类
*   文档进入下一个处理步骤

## 配置

**K 近邻分类器**

<updateRequestProcessorChain name = " class ification ">
<处理器 class="solr。classificationupdateprocessorfactory ">
<str name = " input fields ">title^1.5,content,author</str>
<str name = " class field ">cat</str>
<str name = " algorithm ">KNN</str>

**简单朴素贝叶斯分类器**

<updateRequestProcessorChain name = " classification ">
<处理器 class="solr。classificationupdateprocessorfactory ">
<str name = " input fields ">title^1.5,content,author</str>
<str name = " class field ">cat</str>
<str name = " algorithm ">Bayes</str>

**更新处理器配置**

<request handler name = "/update ">
<lst name = " defaults ">
<str name = " update . chain ">分类</str>
</lst>
</request handler>

| 参数 | 默认 | 描述 |
| 

```
inputFields
```

 | 此配置参数是必需的 | 进行分类时要考虑的字段列表(用逗号分隔)。
每个字段都支持 Boosting 语法。 |
| 

```
classField
```

 | 此配置参数是必需的 | 包含文档类别的字段。它必须出现在索引文档中。
如果 **knn** 算法它必须被**存储。**
如果**贝叶斯**算法，它必须**索引**并且理想情况下**不会被大量分析。** |
| 

```
algorithm
```

 | knn | 用于分类的算法:
–KNN(K 最近邻)
–Bayes(简单朴素贝叶斯) |
| 

```
knn.k
```

 | 10 | *高级*–在 MLT 结果中选择的用于查找最近邻的顶级文档的数量 |
| 

```
knn.minDf
```

 | 1 | *高级*–只有在索引中至少出现这一最小数量的文档时，算法才会考虑某个术语(来自输入文本) |
| 

```
knn.minTf
```

 | 1 | *高级*–一个术语(来自输入文本)只有在输入中出现至少这个最小次数时，算法才会考虑它 |

## 使用

索引新闻文档？我们可以使用已经编入索引的新闻类别，在没有人工干预的情况下自动标记即将发生的故事。
电子商务搜索系统？在用手动分配类别索引了有效的初始产品集之后，类别分配将需要很少的人工交互。
这个更新请求处理器的可能用途数不胜数。
在任何情况下，如果我们的搜索系统中有手动指定类别或分类的文档，自动分类会是一个完美的选择。
利用现有的索引，分类处理的开销将是最小的。
在最初人类努力获得一个好的分类文档语料库之后，搜索系统将能够为即将到来的文档自动索引类别。
当然，我们必须记住，对于需要深度调优的高级分类场景，这种解决方案可能不是最佳的。

## 密码

The patch is attached to this Jira Issue :*[SOLR-7739](https://web.archive.org/web/20220930000700/https://issues.apache.org/jira/browse/SOLR-7739)*This has been officially merged to Apache Solr starting with 6.1\. version.// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930000700/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220930000700/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930000700/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于索引时 Solr 文档分类的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！