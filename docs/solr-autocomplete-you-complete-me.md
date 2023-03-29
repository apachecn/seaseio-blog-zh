# 你让我完整了！Apache Solr 自动完成

> 原文：<https://web.archive.org/web/sease.io/2015/07/solr-autocomplete-you-complete-me.html>

这篇博客文章是关于 Apache Solr 自动完成功能的。
很明显，wiki 上现有的文档还不足以完全理解 Solr Suggester:这篇博文将通过例子、技巧和提示描述所有可用的实现。

## 介绍

如果说几个月的 Solr 用户邮件列表教会了我一件事的话，那就是搜索引擎中的自动完成功能是至关重要的，围绕 Apache Solr Suggester 的宣传和困惑一样多。

在这篇博客中，我将尽可能地澄清所有可以在 Solr 中使用的建议，详细探索它们是如何工作的，并展示一些真实世界的例子。

详细探讨这些配置不在这篇博文的讨论范围之内。

请使用官方维基【T3【1】整合此资源。

让我们从 Apache Solr Suggester 组件的定义开始。

## 阿帕奇 Solr 建议者

来自官方维基[【1】](https://web.archive.org/web/20220928111248/https://solr.apache.org/guide/6_6/suggester.html):

” The SuggestComponent in Solr provides users with automatic suggestions for query terms. You can use this to implement a powerful auto-suggest feature in your search application.This approach utilizes Lucene’s Suggester implementation and supports all of the lookup implementations available in Lucene.The main features of this Suggester are:

*   *   *   **查找实现**可插拔性
        *   **术语词典可插入性**，让您灵活选择词典实现
        *   **分布式支持**

关于配置参数的细节，我建议你去官方 wiki 作为参考。

我们的重点将放在不同查找实现的实际使用上，并给出清晰的例子。

## 术语词典

The Term Dictionary defines the way the **terms (source for the suggestions)** are retrieved for the Solr suggester.There are different ways of retrieving the terms, we are going to focus on the **DocumentDictionary** ( the most common and simple to use).For details about the other Dictionaries implementation please refer to the official documentation as usual.

**DocumentDictionary** 使用 **Lucene 索引**来提供可能的建议列表，具体来说**一个字段**被设置为这些术语的**源**。

## Solr 建议大楼

Building a suggester is the process of :

*   *   从字典中检索术语(建议的来源)
    *   构建建议者在查询时查找所需的数据结构
    *   将数据结构存储在内存/磁盘中

产生的数据结构将首先存储在内存中。

建议在磁盘上额外存储已构建的数据结构，这样当它不再位于内存中时，无需重新构建即可使用。

例如，当您启动 Solr 时，数据将从磁盘加载到内存中，而不需要进行任何重建。

该参数是:

模糊查找的“**存储目录**

**索引路径**为

The built data structures will be later used by the suggester **lookup** strategy, at **query time**.In detail, for the **DocumentDictionary** during the building process, for **ALL the documents** in the index :

*   *   **从磁盘中读取已配置字段的存储内容**(该字段需要 **stored="true"** 才能使建议器工作)
    *   压缩的内容被解压缩(记住 Solr 存储应用压缩算法的字段的普通内容)[【3】](https://web.archive.org/web/20220928111248/https://www.elastic.co/blog/store-compression-in-lucene-and-elasticsearch)
    *   建立了建议的数据结构

We must be really careful here to this sentence :“for ALL the documents**” -> no delta dictionary building is happening**So extra care every time you decide to build the Solr Suggester!Two suggester configuration are strictly related to this observation :

| 参数 | 描述 |
| buildOnCommit 或 buildOnOptimize | 如果为真，则将在每次软提交后重建查找数据结构。如果**为假**，则**默认为**，那么只有当查询参数 suggest.build=true 请求时，才会构建查找数据。因为前面的观察很容易理解，buildOnCommit **是高度不鼓励的**。 |
| buildOnStartup | 如果为真，那么当 Solr 启动或内核重新加载时，将构建查找数据结构。如果未指定此参数，建议者将检查查找数据结构是否存在于磁盘上，如果没有找到，则构建它。同样，**非常不鼓励**将此设置为真，否则我们的 Solr 内核可能需要很长时间 |

在这一点上，一个很好的考虑是在字典构建中引入一个 delta 方法。

这可能是一个很好的改进，让“buildOnCommit”特性变得更有意义。

我将继续验证该解决方案的技术可行性。

现在让我们用相关的例子来描述各种查找实现。**注意**:当使用字段类型“text_en”时，我们指的是启用了软词干和停止过滤器的简单英语分析器。

示例的简单文档集如下:

```
[
      {
        "id":"44",
        "title":"Video gaming: the history"},
      {
        "id":"11",
        "title":"Video games are an economic business"},
      {
        "id":"55",
        "title":"The new generation of PC and Console Video games"},
      {
        "id":"33",
        "title":"Video games: multiplayer gaming"}]
```

以及一个简单的同义词映射:多人在线

## 分析工厂

`<lst name="suggester">`

< str name="name" >分析建议者< /str >

<str name = " lookup impl ">AnalyzingLookupFactory</str>

<str name = " dictionary impl ">documentationary factory</str>

<str name = " field ">title</str>

<str name = " weight field ">价格< /str >

<str name = " suggestAnalyzerFieldType ">text _ en</str>

< /lst >

|   | 描述 |
| **数据结构** | 平面电视 |
| **建筑** | 对于每个文档，根据 *suggestAnalyzerFieldType 对来自字段的**存储内容**进行**分析**。*产生的令牌被添加到索引 FST。 |
| **查找策略** | 分析查询，将产生的标记添加到查询 FST 中。索引 FST 和查询 FST 之间发生交集。建议从字段内容的**开始**开始标识。 |
| **建议返回** | 字段的整个内容。 |

这个建议器非常强大，因为它允许在字段内容的开头提供建议，利用了字段提供的分析链。

通过这种方式，可以提供考虑到**同义词**、**停用词、词干**和分析中使用的任何其他标记过滤器的建议。让我们看一些例子:

| 自动完成的查询 | 建议 | 说明 |
| *“视频游戏”* | 

*   "**视频游戏** ing:历史"
*   “视频游戏是一项经济业务”
*   "**视频游戏** e:多人游戏"

 | 出现的建议只是前缀匹配的结果。目前没有惊喜。 |
| *【电子游戏】* | 

*   **电子游戏**:历史
*   视频游戏是一项经济业务
*   **视频游戏**:多人游戏

 | 对输入查询进行分析，产生的令牌如下:**“视频”“游戏”**。构建时也应用了该分析，为标题的开头产生了相同的**词干术语**。【视频游戏】-->**《视频游戏》**【视频游戏】-->**《视频游戏》**因此前缀匹配适用。 |
| *《电子游戏经济学》* | 

*   "T39 电子游戏是一项经济业务"

 | 在这种情况下，我们可以看到在构建索引 FST 时没有考虑**停用词**。注意:必须不保留位置增量，此示例才能工作，请参见配置详细信息。 |
| *《电子游戏 online ga》* | 

*   **电子游戏:多人游戏**明

 | **同义词扩展**已经发生，匹配返回为在线，多人被建议者认为是同义词，基于应用的分析。 |

## 模糊 LookupFactory

`<lst name="suggester">`

<str name = " name ">fuzzy suggester</str>

<str name = " lookup impl ">FuzzyLookupFactory</str>

<str name = " dictionary impl ">documentdictionary factory</str>

<str name = " field ">title</str>

<str name = " weight field ">价格< /str >

<str name = " suggestAnalyzerFieldType ">text _ en</str>

< /lst >

|   | 描述 |
| **数据结构** | 平面电视 |
| **建筑** | 对于每个文档，根据 *suggestAnalyzerFieldType 对来自字段的**存储内容**进行**分析**。*产生的令牌被添加到索引 FST。 |
| **查找策略** | 对查询进行分析，然后扩展生成的标记，根据为字符串距离函数配置的最大编辑为每个标记生成所有变化(默认为 Levestein 距离[【4】](https://web.archive.org/web/20220928111248/https://en.wikipedia.org/wiki/Levenshtein_distance))。最终产生的标记被添加到查询 FST，保持变化。索引 FST 和查询 FST 之间发生交集。建议从字段内容的**开始**处开始标识。 |
| **建议返回** | 字段的**全部内容**。 |

这个建议器非常强大，因为它允许在字段内容的开头提供建议，利用了字段提供的分析链顶部的模糊搜索。

以这种方式，将有可能提供考虑到**同义词**、**停用词、词干**和在分析中使用的任何其他标记过滤器的建议，并且还支持用户拼写错误的术语。

它是分析查找的扩展。**重要提示**:记住查询时发生的正确处理顺序:

*   *   **首先是**，对**的查询进行分析**，并产生令牌
    *   **然后是**，根据编辑距离和配置的距离算法，用变音扩展记号

**让我们看一些例子:**

 **| 自动完成的查询 | 建议 | 说明 |
| *“视频 gmaes”* | 

*   **电子游戏**:历史
*   视频游戏是一项经济业务
*   **视频游戏**:多人游戏

 | 对输入查询进行分析，产生的令牌如下:**“视频”“gmae”**。然后，相关联的 FST 用包含每个令牌的变化的新状态来扩展。例如**“游戏”**将被添加到查询 FST，因为它与原始令牌的距离为 1。前缀匹配运行良好，返回了预期的建议。 |
| *视频 gmaing* | 

*   **视频游戏**游戏:历史
*   "**视频游戏是一种经济业务"**
*   "**视频游戏** e:多人游戏"

 | 输入的查询被分析，产生的标记如下:**“视频”“GMA”**。然后，相关联的 FST 用包含每个令牌的变化的新状态来扩展。例如**“gam”**将被添加到查询 FST，因为它与原始令牌的距离为 1。因此前缀匹配适用。**** |
| *视频游戏* | 

*   未返回任何建议

 | 这乍一看似乎很奇怪，但它与查找实现是一致的。对输入查询进行分析，产生的标记如下:**“video”“gamign”**。然后，相关联的 FST 用包含每个令牌的变化的新状态来扩展。例如**“gaming”**将被添加到查询 FST，因为它与原始令牌的距离为 1。但是没有前缀匹配适用，因为在索引 FST 中我们有**“game”**，即“gaming”的词干标记 |

## 分析 gInfixLookupFactory

`<lst name="suggester">`

<str name = " name ">analyzinginfixsuggest er</str>

<str name = " lookup impl ">analyzinginfixlookup factory</str>

<str name = " dictionary impl ">documentdictionary factory</str>

<str name = " field ">title</str>

<str name = " weight field ">价格< /str >

str name = " suggestAnalyzerFieldType ">text _ en</str>

</lst>

| 描述 |
| **数据结构** | 辅助 Lucene 指数 |
| **建筑** | 对于每个文档，根据*suggestAnalyzerFieldType*对来自字段的**存储内容**进行**分析**，然后再对 *EdgeNgram* 令牌进行过滤*。*最后，用这些记号建立一个辅助索引。 |
| **查找策略** | 根据*建议分析字段类型*对查询进行分析。然后针对**辅助 Lucene 索引**触发短语搜索建议从字段内容中每个标记的**开头开始识别。** |
| **建议返回** | **字段的全部内容**。 |

这种建议器现在非常普遍，因为它允许在字段内容中间提供建议，利用了字段提供的分析链。

通过这种方式，可以提供考虑到**同义词**、**停用词、词干**和分析中使用的任何其他标记过滤器的建议，并基于**内部标记**匹配建议。让我们看一些例子:

| 自动完成的查询 | 建议 | 说明 |
| *【游戏】* | 

*   "视频 **gami** ng:历史"
*   “视频**游戏**是一种经济业务”
*   "视频**游戏**:多人游戏"

 | 对输入的查询进行分析，产生的令牌如下:**“game”。**在辅助索引中，对于每个字段内容，我们都有 EdgeNgram 标记:“v”，“vi”，“vid”…，“g”，“ga”，“gam”，**“游戏”**。所以匹配发生了，建议被返回 |
| *【嘎】* | 

*   《视频**嘎**明:历史》
*   “视频 **ga** mes 是一种经济业务”
*   "视频 **ga** me:多人游戏"

 | 对输入查询进行分析，产生的标记如下:**“ga”。**在辅助索引中，对于每个字段内容，我们都有 EdgeNgram 标记:" v "，" vi "，" vid"…，" g "，" T71 "，" ga "，" T72 "，" gam "，" game "。所以匹配发生了，建议被返回 |
| *《游戏经济》* | 

*   “视频游戏是一项经济业务”

 | 停用词不会出现在辅助索引中。“游戏”和“经济”都是，所以匹配适用。 |

## BlendedInfixLookupFactory

我们不打算描述这个查找策略的细节，因为它与 AnalyzingInfix 非常相似。

唯一的区别是对建议进行评分，对匹配文档中的前缀匹配进行加权。如果命中更接近建议的开始，得分将更高，反之亦然。

## FSTLookupFactory

`<lst name="suggester">`

<str name = " name ">FSTSuggester</str>

<str name = " lookup impl ">FSTLookupFactory</str>

<str name = " dictionary impl ">documentdictionary factory</str>

<str name = " field ">title</str>

< /lst >

|   | 描述 |
| **数据结构** | 平面电视 |
| **大楼** | 对于每个文档，将**存储内容**添加到索引 FST。 |
| **查找策略** | 该查询被添加到查询 FST 中。索引 FST 和查询 FST 之间发生交集。建议从字段内容的**开始**处开始标识。 |
| **建议返回** | 字段的**全部内容**。 |

这个 Solr suggester 非常简单，因为它允许在字段内容的开头提供建议，并带有精确的前缀匹配。让我们看一些例子:

| 自动完成的查询 | 建议 | 说明 |
| *《视频游戏》* | 

*   "**视频游戏** ing:历史"
*   视频游戏是一项经济业务
*   "**视频游戏** e:多人游戏"

 | 出现的建议只是前缀匹配的结果。目前没有惊喜。 |
| *【电子游戏】* | 

*   没有建议

 | 不对输入查询进行分析，文档中没有以该前缀开头的字段内容**** |
| *【视频游戏】* | 

*   没有建议

 | 不对输入查询进行分析，文档中没有以该前缀开头的字段内容 |
| *【游戏】* | 

*   没有建议

 | 这种查找策略只在字段内容的开头起作用。因此不返回任何建议。 |

对于下面的查找策略，我们将使用稍加修改的文档集:

```
[
      {
        "id":"44",
        "title":"Video games: the history"},
      {
        "id":"11",
        "title":"Video games the historical background"},
      {
        "id":"55",
        "title":"Superman, hero of the modern time"},
      {
        "id":"33",
        "title":"the study of the hierarchical faceting"}]
```

## FreeTextLookupFactory

<lst name = " suggest er ">

<str name = " name ">FreeTextSuggester</str>

<str name = " lookup impl ">freetext lookup factory</str>

<str name = " dictionary impl ">documentdictionary factory</str>

<str name = " field ">title</str>

<str name = " ngrams ">3</str>

<str name = " separator "></str>

<str name = " suggesttfreetextanalyzerfieldtype ">text _ general</str>

< /lst >

|   | 描述 |
| **数据结构** | 平面电视 |
| **建筑** | 对于每个文档，根据*suggesttfreetextanalyzerfieldtype 对字段中的**存储内容**进行**分析**。*作为最后一个令牌过滤器，添加了一个具有 minShingle = 2 和 maxShingle =的木瓦过滤器。产生的最终令牌被添加到索引 FST。 |
| **查找策略** | 根据*suggesttfreetextanalyzerfieldtype*对查询进行分析。作为最后一个令牌过滤器，添加了一个具有 minShingle = 2 和 maxShingle =的木瓦过滤器。只有最新的 *"ngrams"* 令牌将被评估以产生 |
| **建议返回** | ngram 令牌建议 |

这种查找策略与迄今为止看到的其他策略完全不同，其主要区别在于**建议是 ngram 令牌**(而不是字段的全部内容)。

我们在使用这个 Solr suggester 时必须格外小心，因为它很容易出错，一些准则:

*   **不要使用沉重的分析器**，建议的术语将来自索引，所以要确保它们是有意义的记号。一个真正基本的分析器被建议，**停止词和词干不是**
*   确保您使用了正确的分隔符(建议使用')，默认值将编码为“# 30；”
*   **ngrams** 参数将设置从查询中考虑的**最后 n 个标记**

让我们看一些例子:

| 自动完成的查询 | 建议 | 说明 |
| *【视频 g】* | 

*   **视频**g**阿明**
*   **视频 g** 艾姆斯"
*   **g** 代

 | 对输入查询进行分析，产生的标记如下:**“视频 g”“g”**这种分析在建造时也应用了，产生了 2-3 块木瓦。**“视频 g”**通过前缀 2 匹配索引 FST 中的瓦片区。**“g”**通过前缀 1 匹配索引 FST 中的瓦片区。 |
| *《游戏人间》* | 

*   "**游戏历史"**
*   "**游戏 h** 历史"
*   **这个 h** 的层次结构
*   " **h** ero "

 | 对输入查询进行分析，产生的标记如下:**“游戏 h”“游戏 h”“游戏 h”**这种分析在建造时也应用了，产生了 2-3 块木瓦。**“游戏 h”****根据索引 FST 中的前缀 2 瓦片匹配。******“h”**通过前缀 1 匹配索引 FST 中的瓦片区。******“h”**通过前缀 1 匹配索引 FST 中的瓦片区。** |

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220928111248/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220928111248/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220928111248/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于 Apache Solr Suggester 的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**