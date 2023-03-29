# Elasticsearch _source、doc_values 和商店绩效

> 原文：<https://web.archive.org/web/sease.io/2021/02/field-retrieval-performance-in-elasticsearch.html>

在这篇博文中，我想从性能的角度探讨一下 elasticsearch 为我们存储字段和在查询时检索字段提供了哪些可能性**。事实上，构建 elasticsearch 和 solr 的底层库 lucene 提供了两种存储和检索字段的方法:存储字段和文档值。此外，elasticsearch 使用 _source 字段作为默认字段，这是一个很大的 json，包含在索引时作为输入给出的文档的所有字段。**

 **为什么 elasticsearch 使用 _source 字段作为默认字段，从性能角度来看，所有这些可能性有什么不同？让我们来了解一下！**  **## lucene 中的存储和文档值字段

当我们在 lucene 中索引一个文档时，关于已经被索引的原始字段的信息会丢失。根据模式配置对字段进行分析、转换和索引。在没有任何附加数据结构的情况下，当我们搜索一个文档时，我们得到的是被搜索文档的 id，而不是原始字段。为了获得这些信息，我们需要额外的数据结构。Lucene 为此提供了两种可能性:存储字段和文档值。

###### 存储字段

存储字段的目的是存储字段的值(不进行任何分析)，以便在查询时检索它们。

###### 文档值

引入文档值是为了加快分面、分类和分组等操作。Docvalues 还可以用于在查询时返回字段值。唯一的限制是我们不能将它们用于文本字段。

存储的字段和文档值在 lucene 库中实现，它们可以在 solr 和 elasticsearch 中使用。

我写过一篇博文，比较了 solr 中存储字段和 docvalues 的字段检索性能:

[文档值与存储字段:Apache Solr 特性和性能对比](https://web.archive.org/web/20220925163558/https://sease.io/2021/02/field-retrieval-performance-in-elasticsearch.html)

在那里，您可以找到关于存储字段和文档值、它们的利用率和约束的更详细的描述。

## 弹性研究中的字段检索

如果我们在映射中明确定义存储的字段和文档值，则它们可以在 elasticsearch 中使用:

`"properties" : {
  "field": {
   "type": "keyword",
    "store": true,
     "doc_values" true
  }
}`

默认情况下，每个字段的 store 都设置为 false。相反，默认情况下，所有支持文档值的字段都会启用它们。

独立于 stored 和 docvalues 配置，在查询时，将返回查询命中的文档中每个字段的值。这是因为 elasticsearch 使用另一个工具进行字段检索:字段**elastic search****_ source**。

###### 弹性搜索 _ 源字段

源字段是索引时传递给 elasticsearch 的 json。该字段在 elasticsearch 中默认设置为 true，可以通过以下方式使用映射禁用:

`"mappings": {
    "_source": {
     "enabled": false
    }
  }`

默认情况下，查询时会返回所有字段。您甚至可以指定在响应中只返回源中字段的子集。这应该可以加快响应在网络中的传输速度。

有些字段可以通过适当的配置被源字段排除:

`PUT logs
{
  "mappings": {
   "_source": {
    "excludes": [
     "meta.description",
     "meta.other.*"
    ]
   }
  }
}`

从源中排除字段将减少磁盘空间的使用，但被排除的字段永远不会在响应中返回。



禁用 elasticsearch _source 字段将无法更新文档，除非从头开始重新编制索引。事实上，为了更新文档，我们需要从旧文档中获取字段的值。从逻辑上讲，使用存储的字段或 docvalues 从旧文档中获取字段值应该是可行的(这就是 solr 中原子更新的工作方式)。但是，由于设计决策的原因，这在 elasticsearch 中是不允许的，如果您需要更新文档，您必须在 elasticsearch 索引配置中启用 _source 字段。

###### 检索字段

在 elasticsearch 中，您可以启用或禁用 _source 字段，并使字段存储和/或文档值。但是如何在查询时检索字段呢？



默认情况下，如果定义了源，则返回整个源。您可以避免它，只返回源的子集，如下所示:

`...
"fields": ["field1", "field2"],
"_source": false
...`

但是，如果您没有启用源字段，并且您想从 stored 或 docvalues 返回字段，您必须以另一种方式告诉 elasticsearch。对于您使用的每个源，您必须以不同的方式指定字段列表:

`...
"fields": ["sv1", "sv2",...],
"docvalue_fields": ["dv1", "dv2",...],
"stored_fields" : ["s1", "s2",...],
...`

例如，如果您有一个既存储又文档值的字段，您可以选择是从文档值还是存储字段中检索它。从功能的角度来看，这是完全一样的，但是您的选择可能会影响查询的执行时间。

###### 存储字段、文档值和 ELASTICSEARCH_SOURCE 内部表示

在本节中，我只想给出一个关于存储字段、_source 字段和 docvalues 的内部内容的边缘概述，以便有一些工具来理解使用这些方法进行字段检索的预期性能。

Stored fields internals

存储的字段以行的方式放在磁盘中:对于每个文档，我们有一行连续包含所有存储的字段。

![](img/33015149f0223152cdd4d184e73819b7.png)

以上图为例。为了访问文档 x 的字段 3，我们必须访问文档 x 的行并跳过存储在字段 3 之前的所有字段。跳过一个字段需要获取它的长度。跳过字段不像读取字段那样昂贵，但是这种操作不是免费的。

Docvalues internals

文档值以列的方式存储。不同文档的相同字段的值被一起连续存储在存储器中，并且可以“几乎”直接访问特定文档的特定字段。计算想要的值的地址不是一个简单的操作，它有计算成本，但我们可以想象，如果我们只想要一个字段，使用这种访问更有效。

ELASTICSEARCH _SOURCE FIELD INTERNALS

那么 _source 呢？嗯，如上所述，source 是一个大字段，其中包含一个 json，包含索引时提供给 elasticsearch 的所有输入。但是，这个字段实际上是如何存储的呢？毫不奇怪，elasticsearch 利用了 lucene 已经实现并提供的机制:存储字段。特别是,_source 字段是行中的第一个存储字段。

![](img/eb0816ea8b7444d9203bf01c82cfd0ca.png)

必须阅读整个源代码才能使用其中包含的信息。如果我们想返回一个文档的所有字段，这个过程直观上是最快的。另一方面，如果我们只需要返回它包含的一小部分信息，那么读取这个巨大的字段可能会浪费计算能力。

## 标杆管理

为了对 3 种类型的字段进行基准测试，我在 elasticsearch 中创建了 3 个不同的索引。我已经索引了来自维基百科的 100 万个文档，对于每个文档，我用三种不同的方法索引了 100 个 15 个字符的字符串字段:在第一个索引中，我将字段设置为 stored，在第二个索引中设置为 docvalues。在这两个索引中，我禁用了源字段。在第三个索引中，我只是启用了 source 字段。

文档和查询集合取自[https://github.com/tantivy-search/search-benchmark-game](https://web.archive.org/web/20220925163558/https://github.com/tantivy-search/search-benchmark-game)。我使用真实的集合来模拟真实的场景。

执行细节:

*   CPU: AMD RYZEN 3600
*   内存:32 GB

对于每个查询，我请求最好的 200 个文档，并重复测试，从 1 到 100 改变要返回的字段数量(在我创建的 100 个随机字符串字段中)。

这是基准测试的结果:

![](img/a00bc0bcd1cd5cb2879b16241b1a0c70.png)

结果完全符合我们的预期。**如果我们需要每个文档**有几个字段，建议使用 Docvalues。另一方面，当我们想要返回整个文档时， **_source 字段是最好的字段，存储字段的使用是其他两个字段之间的完美折衷。**

在我所执行的基准测试场景中，如果我们只需要一个字段，那么 **docvalues 的速度几乎是 _source 字段** **的两倍，相反，如果我们想要返回所有字段，那么图表显示当我们使用 _source 字段代替 docvalues 时，速度几乎提高了 2 倍。**



最后，性能不是我们必须考虑的唯一参数。正如我们在这篇博文中简要解释的那样，使用这种或那种方法都有一些限制。可能是因为您的用例的一些约束，您被迫使用这三个中的一个。而且即使从表现来看，我们也没有一个明确的赢家。



如果磁盘空间不成问题，**您甚至可以混合使用不同的方法，将一个字段设置为 stored 和 docvalue，并使源保持启用状态**。在查询时，elasticsearch 允许您选择所需的字段列表，以及是否希望它们从 _source、stored 或 docvalues 返回。** **// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Elasticsearch 初学者](https://web.archive.org/web/20220925163558/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)和 [Apache Solr 初学者](https://web.archive.org/web/20220925163558/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)培训吗？
我们也提供这些话题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220925163558/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Elasticsearch _source、doc_values 和商店绩效的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**