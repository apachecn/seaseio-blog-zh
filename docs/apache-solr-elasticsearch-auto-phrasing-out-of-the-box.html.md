# Apache Solr/Elasticsearch:如何管理开箱即用的多术语概念？

> 原文：<https://web.archive.org/web/sease.io/2018/07/apache-solr-elasticsearch-auto-phrasing-out-of-the-box.html>

这篇 flash 博客文章将解决一个非常具体和常见的问题:如何在一个普通的 Apache Solr/Elasticsearch 实例中管理由多个术语组成的实体/概念(无需安装插件或扩展)。

## (部署)上下文

一个 **Elasticsearch** 或 **Apache Solr** 基础设施，在那里**不能安装**第三方组件(例如插件、过滤器、查询解析器)。发生这种情况有几个原因:

*   *   **内生因素**:缺乏在基础设施中实施/安装所需的专业知识/技能。
    *   **外部因素**:搜索功能存在于一个不允许定制组件的外部管理环境中。例如，亚马逊弹性搜索服务[【1】](https://web.archive.org/web/20220930003944/https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-supported-plugins.html)就是这种情况。

## 问题是

如何对多术语概念(即由多个术语组成的概念)进行建模？

概念是特定领域词汇的基础:“美利坚合众国”、“电话号码”、“超出保修范围”只是您可能想要作为一个整体来管理的实体的例子；如果用户搜索类似“我如何转移电话号码？”你**可能**不想返回关于号码或电话的东西，它们的范围更广(即为了回忆而牺牲精确性)。

注意，“**可能是**”是粗体的，因为这里没有绝对的真理:一切都取决于应用程序运行的功能上下文。这里我们假设这个需求，但是在另一个上下文中事情可能会不同。

## 索引/查询时间解决方案

可以使用两种不同的方法来解决这个问题，这两种方法涉及索引时间和查询时间配置:

*   *   **简单缩写**:使用带有简单缩写[【2】](https://web.archive.org/web/20220930003944/https://www.elastic.co/guide/en/elasticsearch/guide/current/multi-word-synonyms.html#_use_simple_contraction_for_phrase_queries)[【3】](https://web.archive.org/web/20220930003944/https://lucene.apache.org/core/6_6_0//analyzers-common/org/apache/lucene/analysis/synonym/SolrSynonymParser.html)的同义词过滤器，以便注入一个表示概念的**单个术语**(可选同义词)

```
Multimedia Messaging Service,Multimedia Text Message => mms
```

*   *   **瓦片区**:组合**瓦片区**和 **keep 词过滤器**，用于从给定文本中生成二元和三元模型(假设我们只对最多由 3 个术语组成的概念感兴趣)

## ...一个额外的约束:仅限查询时间

第一组(索引+查询时间)的缺点之一是当同义词或关键字列表中发生变化时，**重新索引整个数据集**的成本。所以我们将引入的**附加约束**是:我们希望能够在运行时**改变概念列表，而不需要**任何重新索引。

同样，这不是一个绝对的约束:在很多情况下，重新索引一切都是完全可以的。根据我的经验，这与索引大小以及整个重新索引过程有直接关系。

换句话说:如果完整的重新索引过程花费了合理的时间，并且没有产生任何服务中断，那么您应该考虑取消“仅查询时间”约束。

## 一个解决方案

Elasticsearch 和 Apache Solr[【4】](https://web.archive.org/web/20220930003944/https://lucidworks.com/2017/04/18/multi-word-synonyms-solr-adds-query-time-support/#footnote1)中的同义词管理得到了增强，引入了“图形感知”令牌过滤器，以实现对**多词同义词的全面支持。**

在此之前，索引/查询时间和查询时间方法在处理同义词时都受到一些限制[【5】](https://web.archive.org/web/20220930003944/https://opensourceconnections.com/blog/2013/10/27/why-is-multi-term-synonyms-so-hard-in-solr/)。

无论如何，当前的实现允许我们从整体上正确地管理多词同义词。因此，回到我们的问题，我们也可以尝试随意“塑造”同义词过滤器来管理复合概念。

首先，复合概念可以有也可以没有同义词。

###### 带有同义词的概念

如果概念有一个或多个同义词，我们就在同义词过滤器的常规上下文中。下面是 *synonyms.txt* 文件的示例内容:

```
*Multimedia Messaging Service,Multimedia Text Message,MMS*
*USA,United States of America*
*...*

```

下面是相应的配置:

![](img/93a2ea724c85ac8e2261ca3b95e71f41.png "Apache Solr")

```
<fieldtype name="txt" 
           class="solr.TextField" 
 autoGeneratePhraseQueries="true">
       <analyzer type="index">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
       </analyzer>
       <analyzer type="query">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
           <filter 
              class="solr.SynonymGraphFilterFactory" 
              synonyms="synonyms.txt" 
              expand="true"/>
       </analyzer>
</fieldtype>
<field name="title" type="txt" indexed="true" stored="true"/>
```

![](img/366c9eee6f8e45c8c411288250709db9.png "Elasticsearch")

```
"analysis": {
    "filter": {
      "english_synonyms": {
         "type": "synonym_graph",
         "synonyms_path": "synonyms.txt",
         "expand": true
      }
    },
    "analyzer": {
      "text_index_analyzer": {
        "tokenizer": "standard",
        "filter": [ "lowercase" ]
      },
      "text_query_analyzer": {
        "tokenizer": "standard",
        "filter": [
          "lowercase",
          "english_synonyms"
        ]
      }
    }
...
"properties": {
   "title": {
      "type": "text",
      "analyzer": "text_index_analyzer",
      "search_analyzer": "text_query_analyzer"
   }
}
```

发出以下查询:

![](img/93a2ea724c85ac8e2261ca3b95e71f41.png "Apache Solr")

```
q={!lucene}multimedia messaging service&sow=false&df=title
```

![](img/366c9eee6f8e45c8c411288250709db9.png "Elasticsearch")

```
{
  "query": {
     "match": {
       "title": "Multimedia messaging service"
     }
   }
}

{
   "query": {
     "query_string": {
       "query": "Multimedia messaging service",
       "default_field": "title",
     }
   }
}
```

产生(在 Solr 或 _validate/query 中使用 debug=true？explain=true in Elasticsearch)以下查询:

```
(PhraseQuery(title:"multimedia text message") 
 title:MMS 
 PhraseQuery(title:"multimedia messaging service"))
```

这正是我们想要的:查询中表达的概念已经被检测到，输出查询正在寻找它的扩展(美国)或收缩(美国)形式。
目前为止，一切顺利。

###### 没有同义词的概念

如果一个多术语概念没有任何变体/同义词，但我们仍然希望将其作为一个整体来管理，该怎么办？我们可以在没有同义词的情况下使用同义词过滤器吗？让我们试着看看会发生什么。

第一个显而易见的想法是在我们的同义词中声明如下内容:

```
*Multimedia Messaging Service*  
*...*
```

即包含我们的概念而没有任何同义词的一行。不幸的是，这不起作用，引擎检测到没有同义词，结果查询如下:

```
title:multimedia title:messaging title:service
```

如果在索引链中放置一个虚拟同义词，然后通过停用词过滤器将其删除，也会发生同样的情况。类似于:

```
*Multimedia Messaging Service, **something_that_will_be_configured_as_stopword*** 
*...*
```

所以最后一个机会是复制这个条目，就像这样:

```
*Multimedia Messaging Service, **Multimedia Messaging Service*** 
*...*
```

事情开始变得有趣了。运行上述相同的查询，我们得到以下解释:

![](img/93a2ea724c85ac8e2261ca3b95e71f41.png "Apache Solr")

```
(PhraseQuery(title:"multimedia messaging service") 
 PhraseQuery(title:"multimedia messaging service"))
```

![](img/366c9eee6f8e45c8c411288250709db9.png "Elasticsearch")

```
title:"multimedia messaging service"^2.0
```

```
4.544185 = sum of:
  4.544185 = weight(title:"multimedia messaging service" in 1) [SchemaSimilarity], result of:
    4.544185 = score(doc=1,freq=1.0 = phraseFreq=1.0
), product of:
      2.0 = boost
      2.7725887 = idf(), sum of:
        0.6931472 = idf, computed as ... from:
          1.0 = docFreq
          2.0 = docCount
        0.6931472 = idf, computed as ... from:
          1.0 = docFreq
          2.0 = docCount
        0.6931472 = idf, computed as ... from:
          1.0 = docFreq
...
```

也就是说，上面这场比赛的分数本来应该是 **2.2720926** (4.544185 / 2)，但是因为我们有了这个人为的提升，分数已经翻倍了；同时，强调我们的概念作为一个整体得到了正确的管理是很重要的，上面的查询不会返回与消息、多媒体或服务相关的项目，这些是更广泛的概念。

助推因素是个问题吗？这实际上取决于您的应用程序:您应该

*   *   具有代表性的搜索案例数量
    *   尝试简单的以词为中心的搜索
    *   尝试上面建议的方法
    *   使用[搜索质量评估工具](https://web.archive.org/web/20220930003944/https://sease.io/2018/07/rated-ranking-evaluator.html)
    *   比较和选择

## 摘要

*   *   你有一个 **Elasticsearch** 或者 **Apache Solr** 集群
    *   您**无法安装**自定义插件
    *   你想管理**复合概念**
    *   当添加/删除/更新概念列表时，您**不想重新索引您的语料库**
    *   如果一个概念**有一个或多个同义词**，这很简单:在查询时使用同义词(图)过滤器*
    *   如果一个**概念没有任何同义词**，您仍然可以使用同义词(图形)过滤器:**只需将概念定义增加一倍，**但是要记住应用于相应短语查询*的 double (2.0)提升

** Apache Solr 用户:确保目标字段类型的 autoGeneratePhraseQueries 设置为 true，并且 sow 参数(在 RequestHandler 设置中定义或作为请求参数)也设置为 true。*

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930003944/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220930003944/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930003944/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于 Apache Solr/Elasticsearch 的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！