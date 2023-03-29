# 还是同义词+停用词？？妈妈咪呀！

> 原文：<https://web.archive.org/web/sease.io/2018/08/still-synonyms-stopwords-mamma-mia.html>

## 背景

简要回顾一下我们在上一篇文章中到达[的位置:我们有以下同义词和停用词设置:](https://web.archive.org/web/20221225133945/https://sease.io/2018/07/combining-synonyms-and-stopwords.html)

*   *   同义词= { "超出保修期"，" oow"}
    *   停用字词= {"of"}

这两个过滤器**都是在查询时**专门配置的；首先是同义词过滤器，然后是停用词过滤器。

使用内置的 *StopFilter* 我们有一个同义词检测问题，因为查询字符串中“of”术语的删除(例如“我的设备运行了保修的*)。出于这个原因，我们引入了一个自定义的 *StopFilter* 子类，它能够识别同义词中的停用词。*

我们将要描述的另一个场景略有不同:假设我们有以下数据:

*   *   同义词= {测试代码，tdd，测试}
    *   停用词= {我的，你的，怎么样，到，在}

还是在这里，我们希望在查询时间只管理同义词和停用词。
我们已将本文件编入索引:

```
   {
      "id": 1,
      "title": "Java programmer: do you want to test your code?"
   }
```

像这样的查询:

```
"how to test code in Java?"
```

## 问题:缺少同义词匹配

查询解析器匹配查询中的“测试代码”同义词，并生成如下查询:

```
(title:tdd title:testing PhraseQuery(title:"test code")) title:java
```

不幸的是，没有匹配，因为文档标题包含一个入侵者:在“测试”和“代码”之间的“您的”术语。

## 一种解决方案:带或不带同义词短语的不可见查询

在前一篇文章中，我们强调了**自动生成系列**标志的作用。它负责为所有检测到的多术语同义词创建短语子句。如果这个标志被设置为 false(或者甚至是 missing ),那么生成的查询将不会有任何短语，即使检测到一个多术语同义词。

虽然这通常不是您所期望的，但在这个特定的情况下，它可能是处理这种不匹配的有效替代方法:第一个请求需要“同义词分析”行为，第二个请求则不需要。第一个查询是:

```
(title:tdd title:testing PhraseQuery(title:"test code")) title:java
```

收到空响应后，将发送第二个查询，目标是与字段类型相关的另一个(类似)字段，该字段类型的**autogeneratethersequeries**参数将被设置为 **false** 。这将生成以下查询:

```
(title:testing title:tdd (+title:test +title:code)) title:java
```

在这里我们会得到一个匹配！

几个注意事项:

*   *   在第二次尝试中，我们要求这两个术语(“测试”和“代码”)以任意顺序、任意接近度不相交地出现，因此增加的召回可能会产生一些意想不到的结果。如果我们使用 edismax 查询解析器，那么一个“pf”参数将有助于提升那些在接近度和术语顺序方面更好地符合输入查询的结果。
    *   我们可以将停止过滤器放在索引时间，但是这违反了前提条件:我们需要纯粹的查询时间管理。

如何实现这样的搜索工作流程？在 Solr 中，我们需要几个字段，第一个正好是我们在[前一篇文章](https://web.archive.org/web/20221225133945/https://sease.io/2018/07/combining-synonyms-and-stopwords.html)中描述的字段+字段类型，第二个类似，唯一的区别是在**autogeneratethersequeries**参数中，它被设置为 **false** :

```
<fieldtype 
       name="text_with_synonyms_phrases" 
       class="solr.TextField" autoGeneratePhraseQueries="true">

       <analyzer type="index">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
       </analyzer>
       <analyzer type="query">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
           <filter class="solr.SynonymGraphFilterFactory" 
                   synonyms="synonyms.txt" 
                   ignoreCase="false" 
                   expand="true"/>
           <filter class="sc.SynonymAwareStopFilterFactory" 
                   words="stopwords.txt" 
                   ignoreCase="true"/>
       </analyzer>
</fieldtype>
```

```
<fieldtype 
       name="text_without_synonyms_phrases" 
       class="solr.TextField" autoGeneratePhraseQueries="false">

       <analyzer type="index">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
       </analyzer>
       <analyzer type="query">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
           <filter class="solr.SynonymGraphFilterFactory" 
                   synonyms="synonyms.txt" 
                   ignoreCase="false" 
                   expand="true"/>
           <filter class="sc.SynonymAwareStopFilterFactory" 
                   words="stopwords.txt" 
                   ignoreCase="true"/>
       </analyzer>
</fieldtype>

<field 
      name="title_with_synonyms_phrases" 
      type="text_with_synonyms_phrases .../> <field 
      name="title_without_synonyms_phrases" 
      type="text_without_synonyms_phrases .../>
```

下面是最小的请求处理程序:

```
<requestHandler name="/search" class="solr.SearchHandler" default="true">
       <lst name="defaults">
           <bool name="sow">false</bool>
           <str name="df">title_with_synonyms_phrases</str>
           <str name="defType">lucene</str> 
       </lst>
   </requestHandler>
```

客户端将首先发送这样的请求:

```
/search?q=how to test code in Java
```

在收到空响应后，它将发送第二个查询:

```
/search?q=how to test code in Java&df=text_without_synonyms_phrases
```

另一个选项是我们的[CompositeRequestHandler](https://web.archive.org/web/20221225133945/https://sease.io/2018/03/compositesearchhandler.html)[【1】](https://web.archive.org/web/20221225133945/https://github.com/SeaseLtd/composite-request-handler)，这是一个 Solr 组件，它以链的形式调用一组 RequestHandler 实例:将调用第一个请求处理程序，目标是**title _ with _ synonyms _ phrases**，如果没有结果，相同的查询将被发送到另一个请求处理程序，目标是**title _ with _ synonyms _ phrases**。

*elastic search 用户注意:在应用上述内容时，您会发现一些不同。虽然***auto _ generate _ phrase _ queries***属性也存在于 Elasticsearch 中，但它没有相同的效果。你要找的是一个与字段类型无关的属性，是一个查询属性*[【2】](https://web.archive.org/web/20221225133945/https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-synonyms)[【3】](https://web.archive.org/web/20221225133945/https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html)*，名为**auto _ generate _ synonyms _ phrase _ query**。*

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20221225133945/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20221225133945/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20221225133945/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于同义词+停用词的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！