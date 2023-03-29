# 同义词和停用词:Vademecum

> 原文：<https://web.archive.org/web/sease.io/2018/09/synonyms-and-stopwords-vademecum.html>

在本帖中，我们将讨论另外两个同义词的情况，并且我们将尝试以一种精确的形式总结所有以前的技巧。沿袭之前帖子[【1】](https://web.archive.org/web/20220929230116/https://sease.io/2018/07/apache-solr-elasticsearch-auto-phrasing-out-of-the-box.html)[【2】](https://web.archive.org/web/20220929230116/https://sease.io/2018/07/combining-synonyms-and-stopwords.html)[【3】](https://web.archive.org/web/20220929230116/https://sease.io/2018/08/still-synonyms-stopwords-mamma-mia.html)的做法，一切都可以同时适用于 **Apache Solr** 和 **Elasticsearch** 。

## 前提

*   *   **查询时的同义词和停用词**:这不仅仅是一个“理论上的”约束；想象一下，如果您必须管理一个属于具有许多中小型索引的同一客户的部署上下文:您不能在每次同义词或停用词发生变化时从头开始重新构建所有内容。
    *   **同义词，而不是上义词或下义词**:或者更好的说法是，我们不是在讨论同义词词典所说的更宽泛、更狭窄或相关的术语。虽然下面的一些内容在这些上下文中也是有效的，但上位词、下位词或相关概念引入的更宽或更窄的范围可能会对评分阶段产生一些奇怪的副作用。

## 测试数据

先说测试数据。

*   *   **同义词** = ["超出保修期，oow "，"转接电话号码，端口号"]
    *   **停用词** = ["of "，" my"]
    *   **查询分析器** = [ "standard_tokenizer "、"小写过滤器"、"同义词(图形)过滤器"、"停用词过滤器"]

## #1:我如何定义多术语概念？

如果您想从整体上管理一个多术语概念，不管它有没有同义词，您都可以使用同义词文件。这里有几个例子:第一个是有一个同义词的概念，第二个没有任何同义词:

```
Multimedia Messaging Service,Multimedia Text Message,MMS
Apache Cassandra, Apache Cassandra
```

如您所见，当一个概念没有任何可用的同义词时，我们可以重复它。

仅限 Solr 用户:不要忘记以下事情:

*   *   请求处理器应该使用 edismax 或 lucene 查询解析器，并且必须将 **SplitOnWhiteSpace** 标志( **sow** )设置为 **true**
    *   包含同义词图表过滤器的字段类型必须将**自动生成系列**设置为**真**

你可以在这里阅读更多关于这种方法的内容。

注意:这将一直工作到 Lucene *SynonymMap* 使用*列表/数组*来收集与给定概念相关的同义词。当并且如果实现将切换到类似 *Set 的方法*，这个技巧很有可能会停止工作。

这是一个痛苦的故事。但他告诉我们，他是个好人，他是个好人。

## #2:如果查询包含带停用词的多术语概念，该怎么办？

想象这样一个查询

```
q=my car is *out **of** warranty.* What can I do?
```

那么，使用上面的配置，在同义词检测之后删除停用词会对生成的查询产生奇怪的影响:“what”术语被错误地添加到同义词短语查询中:“out？保修什么”。

虽然这个问题影响到了*FilteringTokenFilter*(*stop filter*的超类)，因此它的范围更广，但是对于这个特定的问题，我们提出了一个解决方案[【2】](https://web.archive.org/web/20220929230116/https://sease.io/2018/07/combining-synonyms-and-stopwords.html)，它由一个专门的 *StopFilter* 组成，这个解决方案知道同义词标记。结果是，作为先前检测到的同义词的一部分的术语不会被删除，即使它们是停用词。我们字段的查询分析器变成了这样:

```
<tokenizer class="solr.StandardTokenizerFactory"/>
<filter class="solr.LowerCaseFilterFactory"/>
<filter class="solr.SynonymGraphFilterFactory" 
        synonyms="synonyms.txt" 
        ignoreCase="false" 
        expand="true"/>
<filter class="io.sease.SynonymAwareStopFilterFactory" 
        words="stopwords.txt" 
        ignoreCase="true"/>
```

## #3:如果文档包含带有“入侵者”停用词的多术语概念怎么办？

我们有这样一份文件:

```
{
  "id": 1,
  "title": "how do I *transfer* my *phone number*?"
}
```

和查询:

```
q=*transfer phone number* procedure
```

在查询时，同义词被正确地检测到，短语子句被生成，但不幸的是，它与上面的文档不匹配，因为中间的“my”停用词:

您可以在此处阅读[【3】](https://web.archive.org/web/20220929230116/https://sease.io/2018/08/still-synonyms-stopwords-mamma-mia.html)该场景的建议解决方案，它基本上由两步查询计划组成:在第一步中，检测到的同义词生成短语子句，而在第二步中，它们在术语子句中被解构。

## #4:如果查询包含带有“入侵者”停用词的多术语概念，该怎么办？

而我们现在的情况正好相反。我们有这样一份文件:

```
{ 
  "id": 1, 
  "title": "*transfer phone number* procedure" 
}
```

和查询:

```
q=how do I *transfer* my *phone numbe*r?
```

正如您所看到的，在查询时没有检测到同义词，因为术语之间有“my”停用词。虽然上面的文档可能仍然是生成的查询的响应的一部分，但是这里我们将重点放在丢失同义词的检测上。

一个可能的解决方案是在停用字词过滤器前后加倍同义词过滤器:

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
                   ignoreCase="true" 
                   expand="true"/>
           <filter class="io.sease.SynonymAwareStopFilterFactory" 
                   words="stopwords.txt" 
                   ignoreCase="true"/>
           <filter class="solr.SynonymGraphFilterFactory" 
                   synonyms="synonyms.txt" 
                   ignoreCase="true" 
                   expand="true"/>
       </analyzer>
</fieldtype>
```

在第一次迭代中，同义词没有被检测到，然后*停止过滤器*移除“我的”停止字，因此在第二次迭代中，同义词将被正确识别。注意 *StopFilter* 仍然是我们在#2 中介绍的自定义类，因为我们也想涵盖那个场景。

这种方法的缺点是什么？这在我的具体案例中是有效的，但是请注意， *SynonymGraphFilter* 文档中有明确的警告:

注意:这不能消耗传入的图形；结果将是不确定的。

## #5(未解决)如果查询包含多个术语概念，不止一个“入侵者”停用词，该怎么办？

这是最坏的情况，我们有这样一个查询:

```
q=out of my warranty
```

也就是说:我们有两个术语被声明为停用词，但是第一个(of)可能是同义词的一部分(超出保修范围)，而第二个(my)不是。

我们仍在处理这个案子，所以很遗憾这里没有提议，如果你有一些想法或反馈，我们热烈欢迎。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929230116/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929230116/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929230116/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于同义词+停用词的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！