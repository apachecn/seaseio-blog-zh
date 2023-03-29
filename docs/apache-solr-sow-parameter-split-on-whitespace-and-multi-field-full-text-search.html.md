# Apache Solr sow 参数(按空白分割)和多字段全文搜索

> 原文：<https://web.archive.org/web/sease.io/2021/05/apache-solr-sow-parameter-split-on-whitespace-and-multi-field-full-text-search.html>

## eDismax sow 参数如何工作

sow(空格分割)是一个 eDismax 查询解析器参数[【1】](https://web.archive.org/web/20221226202743/https://solr.apache.org/guide/8_8/the-extended-dismax-query-parser.html#extended-dismax-parameters)，它管理查询时文本分析的各个方面，这些方面影响用户查询的解析和内部 Lucene 查询的构建。
这在多术语和多领域搜索中特别相关。
如果**母猪=真**:

*   *   首先，**用户查询文本用空格分割**方法进行标记化
    *   然后，对于令牌流中的每个术语，构建 N 个析取子句，搜索中涉及的每个查询字段一个子句
    *   查询时进行文本分析(因此文本分析在每个字段输入一个术语)

例如

【T8

*   *   **字段 1** 有**多项同义词**配置: *英国，英国，英国，伦敦，英国，英国*
    *   **字段 2** 为查询时文本分析配置了**英文词干**

```
sow = true
qf = field1 field2
q = united kingdom
defType = edismax

"parsedquery_toString":"+((field1:united | field2:unit) (field1:kingdom | field2:kingdom))"
```

field1 没有发生多术语同义词扩展，这是意料之中的，因为文本分析链对单个术语调用了两次: **field1:united** 和 **field1:kingdom** 。
field 1 的文本分析链从未见过文本“英国”，因此没有发生同义词扩展。
另一方面，对 field2 的文本分析已经产生了**词干化的术语(field 2:unit**ed =>**field 2:unit)，**其被预期为逐字地应用词干化。

如果 **sow=false** :

*   *   用户查询未被标记化
    *   为搜索中涉及的每个查询字段构建一个包含整个文本的析取子句
    *   查询时进行文本分析(因此，对于每个字段，文本分析接受输入的完整文本)

例如

*   *   **field1** 有**多项同义词**配置:*英国，英国，英格兰，伦敦，英国，英国 *
    *   **字段 2** 为查询时文本分析配置了**英文词干**
    *   两者都有**autogeneratethersequeries = " true "**(这是扩展的多术语同义词所需要的)

```
sow = false
qf = field1 field2
q = united kingdom
defType = edismax

"parsedquery_toString":
"+((field2:unit field2:kingdom) | 
((field1:uk field1:\"united kingdom\" field1:england field1:london field1:british field1:britain)))",
```

field1 发生了多词同义词扩展，这是意料之中的，因为文本分析链被调用过一次，全文: **field1:(联合** **王国**)。
字段 1 的文本分析链分析了文本“英国”,并且该分析链负责同义词扩展。

## 什么时候 sow=false 是绝对必要的

如果您在查询时配置了文本分析链，这需要输入中的完整文本才能正常工作，因为令牌过滤器会将多个连续的令牌转换为新的令牌，您需要避免初始的空白令牌化，因此 **sow=false** 是必需的:
例如

*   *   从多个术语映射的同义词:**英国** = >英国，英国，英格兰，伦敦，英国，英国
    *   木瓦令牌过滤[【2】](https://web.archive.org/web/20221226202743/https://solr.apache.org/guide/8_8/filter-descriptions.html#shingle-filter):**英国** **探索** = >英国，王国探索



如果您的文本分析与空白标记化不兼容，您必须使用 sow= **false**
例如

```
<fieldType name="text_keyword" class="solr.TextField" positionIncrementGap="100">
       <analyzer>
         <tokenizer class="solr.KeywordTokenizerFactory"/>
         <filter class="solr.LowerCaseFilterFactory"/>
       </analyzer>
     </fieldType>
```

使用 sow=true 只会在与索引不匹配的术语上构建多个查询子句

```
sow = true
qf = author_keyword
q = united kingdom
defType = edismax

"parsedquery_toString":
"+((author_keyword:united) (author_keyword:kingdom))"
```

在这种情况下，必须使用**sow = false**

```
sow = false
qf = author_keyword
q = united kingdom
defType = edismax

"parsedquery_toString":
"+(author_keyword:united kingdom)",
```

## 当 sow=false 不够时

如果你的字段类型是一个字符串(没有被分析),那么 sow= **false** 就没用了。这是一个当前的 bug，例如

```
 <field name="author_s" type="string" indexed="true" stored="true" multiValued="false" />
```

使用 sow=true 只会在与索引不匹配的术语上构建多个查询子句

```
sow = true
qf = author_s
q = united kingdom
defType = edismax

"parsedquery_toString":
"+((author_s:united) (author_s:kingdom))"
```

使用 sow=false 也是失败的

```
sow = false
qf = author_s
q = united kingdom
defType = edismax

"parsedquery_toString":
"+((author_s:united) (author_s:kingdom))"
```

## 为什么要设置 sow=true？

考虑到设置 sow=false 的明显优势，为什么这个参数会存在呢？即，为什么以及何时应该将它设置为 true？

以下考虑假设平局参数[【4】](https://web.archive.org/web/20221226202743/https://solr.apache.org/guide/8_8/the-dismax-query-parser.html#the-tie-tie-breaker-parameter)设置为默认值 0。关于该主题的进一步考虑如下。



*   *   如果您的文本分析为所有查询字段生成完全相同数量的标记，您将看不到设置 **sow=true** 或 **sow=false** 的任何差异
    *   如果您的文本分析从输入文本中为每个字段产生不同数量的标记，差异就出现了:如果您希望包含更多查询词(不一定在同一个字段)**的**文档通常得分更高，您应该使用 sow=true。**
        **sow=true** 每个词对得分有一次贡献(如果匹配两次，则取得分最高的字段)
        如果您通常更喜欢匹配更多查询词的文档，那么您应该更喜欢这种策略。
        这是否意味着拥有更多条款将永远赢得更少条款的支持？不，不一定
        因为超级稀有字段中的一个字段的匹配可能仍然占优势，因为不再有坐标因子
        坐标因子用于将每个布尔子句的贡献归一化为 1/n，其中 n 是涉及的查询词的数量。
        **sow=false** 只有一个字段对得分有贡献(得分最高的字段)**
    *   如果要在关键字标记化的字段中同时搜索多个值

## 文档频率

目前哪些因素影响哪个领域得分最高？ [BM25 计分](https://web.archive.org/web/20221226202743/https://en.wikipedia.org/wiki/Okapi_BM25)。
BM25 是 Apache Solr 和 Lucene 使用的评分算法。
该算法使用一个术语的文档频率来基本估计一个术语在其他查询术语中的重要性。
因此，查询术语在语料库中越稀少，对整个查询来说就越重要。
这对于单字段搜索来说很好。
但是在析取查询中使用它会带来一个讨厌的效果:
文档频率用于选择与某个术语(甚至是单个术语)匹配的最佳字段。它以一种反直觉的方式做到了这一点:术语出现得越少的领域被认为是最好的。
后果显而易见:
**文集**
**文档**:漫画期
**字段** : id、标题、英雄、反派

T36

**查询**
**正文**:蝙蝠侠
**查询字段**:英雄、反派

这导致了析取查询:(英雄:蝙蝠侠|反派:蝙蝠侠)
所以每个文档的得分将是潜在**英雄:蝙蝠侠**匹配和潜在**反派:蝙蝠侠**匹配中的最高分。
让我们假设所有其他因素(术语频率、平均文档长度……)是相同的，文档频率将是决定需要排序的每个候选文档的最佳字段匹配的判别式。
目前，它的工作与用户的直觉预期相反:蝙蝠侠是故事中的反派的漫画将会飙升到顶峰。
倒排文档频率用于选择一个术语的最佳字段，而不是多术语查询中最重要的术语。

## 联系参数

平局参数 [[4]](https://web.archive.org/web/20221226202743/https://solr.apache.org/guide/8_8/the-dismax-query-parser.html#the-tie-tie-breaker-parameter) 使所有其他匹配中“最佳”匹配的影响变平。
在极端情况下，tie=1，此时所有匹配的贡献总和为,**sow =真**或**sow =假**变得对评分没有影响(所有多项令牌过滤观察结果保持不变)。

## 毫米(最小应匹配)

sow 参数影响 mm 参数[【6】](https://web.archive.org/web/20221226202743/https://solr.apache.org/guide/8_8/the-dismax-query-parser.html#mm-minimum-should-match-parameter)。
当解析的查询从以术语为中心(sow=true)变为以字段为中心(sow=false 和不同的文本分析)时，mm 意味着两件不同的事情:
匹配的查询术语最少，与哪个字段无关

```
sow = true
mm=2
qf = author subjects_as_same_term
q = united kingdom
defType = edismax

"parsedquery_toString":
"+(((author:united | subjects_as_same_term:united) (author:kingdom | subjects_as_same_term:kingdom))~2)"
```

```
"response":{"numFound":2,"start":0,"maxScore":7.757958,"numFoundExact":true,"docs":[
      {
        "id":"888888",
        "author":"united",
        "subjects":["kingdom"],
        "score":7.757958},
      {
        "id":"77777",
        "author":"united kingdom",
        "score":5.874222}]
  },
```

同一个字段中匹配的查询词最少(即所有需要的查询词必须在其中一个字段中找到)

```
sow = false
mm=2
qf = author subjects_as_same_term
q = united kingdom
defType = edismax

"parsedquery_toString":
"+(((author:united author:kingdom)~2) | 
(((subjects_as_same_term:uk subjects_as_same_term:\"united kingdom\" 
subjects_as_same_term:england subjects_as_same_term:london 
subjects_as_same_term:british subjects_as_same_term:britain))~1))"
```

this(author:United author:kingdom)~ 2 意味着我们需要这两个子句匹配才能有一个好的候选项，与
(subjects _ as _ same _ term:uk subjects _ as _ same _ term:\ " United kingdom " subjects _ as _ same _ term:England subjects _ as _ same _ term:London subjects _ as _ same _ term:British subjects _ as _ same _ term:Britain)1 的析取关系意味着我们至少需要一个子句匹配(因为同义词将两个原始术语扩展为一个)

```
"response":{"numFound":1,"start":0,"maxScore":5.874222,"numFoundExact":true,"docs":[
      {
        "id":"77777",
        "author":"united kingdom",
        "score":5.874222}]
  }
```

## 突出的问题

eDismax 查询解析器是一个工具，提供了许多配置点，但现在它可能变得太复杂了。
让我们总结一下当前突出的问题:

T32

*   *   如果查询字段共享相同的文本分析，并且不涉及多术语令牌过滤，则 sow=true 或 sow=false 没有区别
    *   由于 coord 已经被移除，sow=true 将不再倾向于多项匹配
    *   当在为查询词选择最佳字段时涉及析取时，由于 BM25 倒排文档频率，在平局决胜时总是选择“不太受欢迎”的字段匹配

## 后续步骤

这是一个痛苦的故事。但他告诉我们，他是个好人，他是个好人。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20221226202743/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20221226202743/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20221226202743/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr sow 参数(按空白分割)和多字段全文搜索的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！