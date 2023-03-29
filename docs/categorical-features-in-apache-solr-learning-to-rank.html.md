# Apache Solr 学习排序中的分类特征

> 原文：<https://web.archive.org/web/sease.io/2022/10/categorical-features-in-apache-solr-learning-to-rank.html>

如果您正在阅读这篇博文，是因为您有兴趣了解更多关于 Apache Solr 学习特性排序的知识，特别是分类特性。(我提到过我们提供关于[学习排名](https://web.archive.org/web/20221130080746/https://sease.io/training/learning-to-rank-training)的私人和公共培训吗？)

因此，在这篇博文中，我将向您概述哪些 Apache Solr 特性类型可用于学习对模型进行排序，以及计算它们需要哪些计算工作量。

让我们从 Apache Solr 学习分级特性的概述开始吧！

## 原始分数特征

The original score feature returns the original score that the document had before performing the reranking.

学习排序是对使用传统搜索方法(例如 BM25)完成的查询所返回的前 k 个元素执行的方法。
在加工流水线上我们，因此:

1.  执行传统查询
2.  获取感兴趣的文档，按相关性排序
3.  从此列表中选择前 k 个文档
4.  执行等级重新评分学习

原始分数是由传统查询返回的分数，用于根据相关性对最初的 top-k 候选项进行排序。这是学习分级功能返回的值。

示例配置:

```
{
  "name": "originalScore",
  "class": "org.apache.solr.ltr.feature.OriginalScoreFeature",
  "params": { } 
 }
```

[ [文档](https://web.archive.org/web/20221130080746/https://solr.apache.org/docs/8_7_0/solr-ltr/org/apache/solr/ltr/feature/OriginalScoreFeature.html)

###### 复杂性

这是最简单的特性，因为它包含一个已经计算好的值。Solr 只需要将这个值作为一个特性传递给排序学习模型。

## 价值特征

The value feature allows returning a constant given value for the current document.

在这里，我们可以决定将哪个值分配给特性。该值可以是常量，也可以在运行时从外部传递(efi，即外部特征信息)。大多数情况下，这用于查询级别的功能，因此这些功能的值取决于查询执行时的条件(例如，用户设备、时间、用户位置等)。
举个例子，假设您希望对来自移动设备和桌面设备的搜索进行不同的排序。在 rerank 请求中，可以传入 rq={… efi.userFromMobile=1}，上述特性将为该请求的所有文档返回 1。

示例配置:

```
{
  "name" : "userFromMobile",
  "class" : "org.apache.solr.ltr.feature.ValueFeature",
  "params" : { "value" : "${userFromMobile}", "required":true }
 }
```

[ [文档](https://web.archive.org/web/20221130080746/https://solr.apache.org/docs/8_7_0/solr-ltr/org/apache/solr/ltr/feature/ValueFeature.html)

###### 复杂性

就简单性而言，这是第二个特性，因为我们传递的是该特性应该为该请求的所有文档假定的确切值。

## 字段长度特征

The field length feature returns the length of a field (number of terms occurring in the field value) for the current document.

示例配置:

```
{
  "name": "titleLength",
  "class": "org.apache.solr.ltr.feature.FieldLengthFeature",
  "params": {
     "field": "title"
 } 
}

```

[ [文档](https://web.archive.org/web/20221130080746/https://solr.apache.org/docs/8_7_0/solr-ltr/org/apache/solr/ltr/feature/FieldLengthFeature.html)

###### 复杂性

沿着这个列表往下看，复杂度成本增加了，因为它需要少量的计算。我们确实需要为请求返回的每个文档计算指定字段的长度(这是从索引中提取的)，以便为特性赋值，并在学习排序过程中使用它。

## 字段值特征

The field value feature returns the value of a field in the current document.

示例配置:

```
{
  "name":  "rawHits",
  "class": "org.apache.solr.ltr.feature.FieldValueFeature",
  "params": {
      "field": "hits"
  }
}
```

[ [文档](https://web.archive.org/web/20221130080746/https://solr.apache.org/docs/8_7_0/solr-ltr/org/apache/solr/ltr/feature/FieldValueFeature.html)

###### 复杂性

这里也需要计算。为了计算这个特性，我们必须从请求返回的每个文档中提取所需的字段值。

## Solr 特征

The Solr feature allows you to use any Solr query as a feature. The value of the feature will be the score of the given query for the current document.

参见 Solr 文档中其他可以用作特性的解析器[ [文档](https://web.archive.org/web/20221130080746/https://lucene.apache.org/solr/guide/other-parsers.html) ]。

示例配置:

```
[{ "name": "isBook",
  "class": "org.apache.solr.ltr.feature.SolrFeature",
  "params":{ "fq": ["{!terms f=category}book"] }
}, 
{
  "name": "documentRecency",
  "class": "org.apache.solr.ltr.feature.SolrFeature",
  "params": {
     "q": "{!func}recip( ms(NOW,publish_date), 3.16e-11, 1, 1)"
   }
 }]
```

[ [文件](https://web.archive.org/web/20221130080746/https://solr.apache.org/docs/8_7_0/solr-ltr/org/apache/solr/ltr/feature/SolrFeature.html)

###### 复杂性

这是最复杂的特性。这里我们必须执行一个完整的查询来获得特征值。

## 分类特征

分类特征表示具有一组不同的可能值的对象的属性。在计算机科学中，通常将分类特征的可能值称为枚举。

举个例子，如果我们的对象是一本书，一个分类特征是:

*<本书作者* >与价值观:*奥威尔、福莱特、奥斯汀，其他*

**注意:**很容易观察到，给一个分类特征的值下一个顺序并不会带来任何好处:*奥威尔* < *福利特* < *奥斯汀*没有一般意义。

为了将这些特征与排序学习(机器学习)模型一起使用，需要将它们编码成数值。编码分类特征最直接的方法是**一键编码**。

给定基数 N，我们构建 N-1 个编码的二进制特征。在我们的例子中:
*book _ author _ Orwell*= 0/1
*book _ author _ follet*= 0/1
*book _ author _ Austen*= 0/1
*book _ author _ other*= 0/1

对于每本书，我们将在对应于该书作者的特征上设置 1，在其他特征上设置 0。

###### 学习对特征进行分级

正如你所看到的，到目前为止，还没有一个预定义的 Solr 学习排序特性来管理分类特性。

目前，这只能通过手动编码特征来实现。完成后，我们可以使用上一段中介绍的**值特征**和 **Solr 特征**来表示该特征。
请注意，Solr 功能是最复杂的功能，因此在时间和内存使用方面都是最昂贵的。

**值特征**可用于表示外部传递的特征，并取决于何时以及谁在执行查询(**查询级别特征**)。这里，外部值将取值 1 或 0，如独热编码所示。

示例配置:

```
{
   "store": "ctrstat_store",
   "name": "userFavouriteAuthor_Orwell",
   "class": "org.apache.solr.ltr.feature.ValueFeature",
   "params": {
      "value": "${userFavouriteAuthor_Orwell}",
      "required": false
    } 
}
```

**Solr 特征**，否则，可以用于表示仅依赖于文档本身的特征(**文档级**)。

示例配置:

```
{
   "store": "ctrstat_store",
   "name": "book_author_orwell",
   "class": "org.apache.solr.ltr.feature.SolrFeature",
   "params": {
      "fq": [
         "{!terms f=author_name}Orwell"
        ] 
    } 
}
```

## 摘要

在这篇博文中，我们看到 Solr 公开了几个学习排名的特性。以下是从最简单到最复杂的功能列表:

*   原始乐谱特征*–最简单的*
*   价值特征
*   字段长度特征
*   字段值特征
*   Solr 特性*–最复杂的*

在这个列表中，没有可以直接管理分类特性的特性。这可以通过手动编码特性并使用示例中所示的 Value 和 Solr 特性来实现。

## 未来作品

我们计划贡献一种新型的 Apache Solr learning to rank 特性，它可以自动管理类别，具有类似于 faceting 的机制(从索引中提取类别，并将 features.json 中的一个定义特性分解为多个编码类别)。

你想赞助这个贡献或帮助吗？随时[联系我们](https://web.archive.org/web/20221130080746/https://sease.io/contacts)！

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做[学习排名](https://web.archive.org/web/20221130080746/https://sease.io/learning-to-rank-training)和 [Apache Solr](https://web.archive.org/web/20221130080746/https://sease.io/training/apache-solr-training) 培训吗？我们也提供关于这些主题的咨询，如果你想让你的搜索引擎更上一层楼，请联系我们。

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 学习排序的分类特性的文章吗？不要忘记订阅我们的时事通讯，以便在信息检索世界中保持最新状态！