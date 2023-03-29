# 在 Apache Lucene/Solr 中引入加权同义词

> 原文：<https://web.archive.org/web/sease.io/2020/03/introducing-weighted-synonyms-in-apache-lucene.html>

这篇博文是关于我们对 Apache Lucene/Solr 项目的最新贡献:
介绍了为同义词分配不同权重的能力。
这篇文章旨在帮助那些处理复杂同义词词典的用户，在这些词典中，为每个同义词关联一个数字权重是很重要的，例如，提升在该领域中更重要或更接近原始概念的同义词。

## 贡献

贡献详见以下吉拉官方发布:
-12238[【1】](https://web.archive.org/web/20220925165828/https://issues.apache.org/jira/browse/SOLR-12238)
LUCENE-9171[【2】](https://web.archive.org/web/20220925165828/https://issues.apache.org/jira/browse/LUCENE-9171)

在 Github Pull 请求[【3】](https://web.archive.org/web/20220925165828/https://github.com/apache/lucene-solr/pull/357)中跟踪了代码审查和合并过程

这个新特性将在 Apache Lucene/Solr 8.5 中提供

变化主要发生在 Lucene 端:
—**新的令牌过滤器，**能够提取权重并将其存储为令牌提升属性
—**查询构建**，检查提升属性并在出现时使用它们构建提升查询

这使得 Apache Solr 和 Elasticsearch(即将推出)都可以使用这个贡献。

Solr 方面，这一变化影响了 Solr 基本查询解析器，以兼容同义词查询方式。

## 阿帕奇索尔

###### 配置

启用查询时间加权同义词需要两种配置:

*   *   按照 delimitedBoost 令牌过滤器的语法，在 synonyms.txt 文件**中定义带有关联权重的同义词(如果您愿意，可以使用托管 REST API 来完成这项工作)**

*同义词. txt*

```
tiger, tigre|0.9
lynx => lince|0.8, lynx_canadensis|0.9

leopard, big cat|0.8, bagheera|0.9, panthera pardus|0.85
lion => panthera leo|0.9, simba leo|0.8, kimba|0.75

panthera pardus, leopard|0.6
panthera tigris => tiger|0.99

snow leopard, panthera uncia|0.9, big cat|0.8, white_leopard|0.6
panthera onca => jaguar|0.95, big cat|0.85, black panther|0.65
panthera blytheae, oldest|0.5 ancient|0.9 panthera
```

*   *   在 schema.xml 中定义一个 fieldType，在查询时展开同义词后应用 delimitedBoost 过滤器

*schema.xml*

```
<fieldType name="boostedSynonyms" class="solr.TextField" positionIncrementGap="100"  synonymQueryStyle="as_distinct_terms" autoGeneratePhraseQueries="true">
    <analyzer type="index">
      ...
    </analyzer>
    <analyzer type="query">
      ...
      <filter name="synonymGraph" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
      <filter name="delimitedBoost"/>
    </analyzer>
  </fieldType>
```

**注意事项**。默认情况下“|”用作权重的分隔符，如果您喜欢任何其他字符，可以使用“分隔符”参数:

```
<filter name="delimitedBoost" delimiter="/"/>
```

仅此而已！
现在您已经准备好探索同义词查询扩展的各种工作方式以及如何应用提升

###### 询问时间

在查询时，您为同义词配置的权重将用于构建一个包装同义词的 **boost 查询**。
在评分时，这是一个**乘法因子**，应用于同义词匹配产生的分数。

例如，
给定一个< **query1** ， **document1** >对，其中 document1 是 query 1
**query 1**= title:(老虎或 tigre^0.8)
**score(document 1)**(让我们只考虑 tigre^0.9 分数部分

```
2.5526304 = weight(title:tigre in 14) [SchemaSimilarity], result of:
    2.5526304 = score(freq=1.0), product of:
      0.8 = boost
      6.531849 = idf
      0.4884969 = tf
```

如何解析带有同义词的查询？

Apache Solr 目前支持三种不同的同义词查询样式:

`**as_same_term**`(默认):混合术语文档频率，即`SynonymQuery(tshirt,tee)`，其中每个术语将被视为同等重要，而与它们在信息语料库中的稀有程度无关(混合文档频率)
`**pick_best**`在用 0 平局因子对`Dismax(tee,tshirt)`评分时选择最重要(最稀有的文档频率)的同义词。
`**as_distinct_terms**`将评分偏向包含更多同义词的文档`(pants OR slacks)`。



您可以在 schema.xml 中配置同义词查询样式:

```
 <fieldType name="text_as_distinct" class="solr.TextField" positionIncrementGap="100"  synonymQueryStyle="as_distinct_terms" autoGeneratePhraseQueries="true">
```

根据您选择的同义词查询样式，您将解析不同的查询，为了这篇博文，下面的示例使用了`**as_same_term**` **(默认)**同义词查询样式。
加权同义词兼容所有的同义词查询样式。

###### **单词查询——单词同义词**

 *同义词. txt*

```
tiger, tigre|0.9
```

*查询* =标题:老虎
*解析查询* :
同义词(标题:老虎 **title:tigre^0.9** )

###### **单词查询-多词同义词**

 *同义词. txt*

```
leopard, big cat|0.8, bagheera|0.9, panthera pardus|0.85
```

*查询* =标题:豹子
*解析查询* :
**((标题:“大 cat")^0.8** 或
**)(title:bagheera)^0.9**或
**(标题:“潘塞拉 pardus")^0.85** 或
标题:豹子)

**注意:**支持多术语同义词，权重被解析为短语查询的提升。

###### **多词查询——多词同义词**

*同义词. txt*

```
snow leopard, panthera uncia|0.9, big cat|0.8, white_leopard|0.6
```

*查询* =标题:【雪豹】
*解析后的查询* :
**((标题:“panthera uncia")^0.9** 或
(标题:“大 cat")^0.8 或
)(title:white_leopard)^0.6 或
标题:“雪豹”)

**注意:**支持多术语同义词，权重被解析为短语查询的提升。

###### **原有的助推兼容性**

该贡献与预先存在的提升兼容(例如来自 edismax 查询解析器)。



*同义词. txt*

```
snow leopard, panthera uncia|0.9, big cat|0.8, white_leopard|0.6
```

Edismax *查询* =雪豹
qf = title^10
*解析查询*:
(**(标题:【panthera uncia")^0.9** 或
t38】(标题:【大 cat")^0.8 或
t41】(title:white_leopard)^0.6 或
标题:【雪豹】 **^10.0**

T47

**注意:**支持多术语同义词，权重被解析为短语查询的提升。

## 结论

要改进 Apache Lucene/Solr 中同义词的管理方式，还有很多工作要做，特别是上义词(泛化)和下义词(规范)。这只是第一步🙂

未完待续

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220925165828/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220925165828/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们也提供关于这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220925165828/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Lucene/Solr 中加权同义词的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！