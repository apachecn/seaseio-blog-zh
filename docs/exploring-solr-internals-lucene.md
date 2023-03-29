# 探索 Solr 内部:Lucene 倒排索引

> 原文：<https://web.archive.org/web/sease.io/2015/07/exploring-solr-internals-lucene.html>

## 介绍

这篇博文是关于 Lucene 倒排索引和 Apache Solr 内部如何工作的。

在使用 Solr 系统时，理解并正确配置下划线 Lucene 索引是深度控制搜索的基础。通过更好地了解索引的外观以及每个组件的使用方式，您可以构建一个更高性能、轻量级和高效的解决方案。
这篇博文的范围是探索倒排索引中的不同组件。
这将更多地是关于数据结构以及它们如何有助于提供搜索相关的功能。
关于存储数据结构的底层方法，请参考 Lucene 官方文档[【1】](https://web.archive.org/web/20220929232332/https://lucene.apache.org/core/5_2_1/core/org/apache/lucene/codecs/lucene50/package-summary.html#package_description)。

## Lucene 倒排索引

The Inverted Index is the basic data structure used by Lucene to provide Search in a corpus of documents.It’s pretty much quite similar to the index in the end of a book.From wikipedia :

在计算机科学中，倒排索引(也称为发布文件或倒排文件)是一种索引数据结构，存储从内容(如单词或数字)到其在数据库文件、一个文档或一组文档中的位置的映射

## 在内存中/磁盘上

The Inverted index is the core data structure that is used to provide Search.We are going to see in details all the components involved.It’s important to know where the Inverted index will be stored.Assuming we are using a **FileSystem Lucene Directory**, the index will be stored on the disk for durability ( we will not cover here the Commit concept and policies).Modern implementation of the FileSystem Directory will leverage the OS Memory Mapping feature to actually load into the memory ( RAM ) chunk of the index ( or possibly all the index) when necessary.The index in the file system will look like a collection of **immutable segments.****Each segment is a fully working Inverted Index, built from a set of documents.****The segment is a partition of the full index, it represents a part of it and it is fully searchable.**Each segment is composed by a number of binary files, each of them storing a particular data structure relevant to the index, compressed [[1]](https://web.archive.org/web/20220929232332/https://lucene.apache.org/core/5_2_1/core/org/apache/lucene/codecs/lucene50/package-summary.html#package_description).To simplify, in the life of our index, while we are indexing data, we build segments, which are merged from time to time ( depending of the configured Merge Policy).But the scope of this post is not the Indexing process but the structure of the Index produced.

## 把手举起来。

假设输入 3 个文档，每个文档有 2 个简单的字段，看看完整的倒排索引是什么样子的:

Doc0
{ "id":"c "，
"title ":"电子游戏历史"
}，

Doc1
{ " id ":" a "、
"title ":"游戏视频回顾游戏"
}、

Doc2
{ " id ":" b "，
"title ":"游戏商店"
}，

根据 schema.xml 中的配置，我们在索引时生成相关的数据结构。

让我们看看完整形式的倒排索引，然后让我们解释每个组件如何使用，以及何时省略它的一部分。

| 田 | id |
| 序数 | 学期 | 文档频率 | 发布列表 |
| 0 | 答 | 1 | **1** : 1 : [1] : [0-1] |
| 1 | b | 1 | **2**:1:[1]:[0-1] |
| 2 | c | 1 | **0**:1:[1]:[0-1] |

| 田 | 标题 |
| 序数 | 学期 | 文档频率 | 过帐清单 |
| 0 | 游戏 | 3 | **0**:1:[2]:【6-10】，
**1**:2:[1，4]:【0-4，18-22】，
**2**:1:[0-4] |
| 1 | 历史 | 1 | **0** : 1 : [3] : [11-18] |
| 2 | 回顾 | 1 | **1**:1:[3]:[11-17] |
| 3 | 商店 | 1 | **2**:1:[2]:[5-10] |
| 4 | 视频 | 2 | **0**:1:[1]:【0-5】，
**1**:1:[2]:【5-10】， |

这听起来有点吓人，让我们分析一下数据结构的不同组成部分。

## 术语词典

The term dictionary is a **sorted skip list** containing all the unique terms for the specific field.Two operations are permitted, starting from a pointer in the dictionary :**next()** -> to iterate one by one on the terms**advance(ByteRef b)** -> to jump to an entry >= than the input  ( this operation is O(n) = log n where n= number of unique terms).An auxiliary Automaton is stored in memory, accepting a set of smart calculated prefixes of the terms in the dictionary.It is a weighted Automaton, and a weight will be associated to each prefix ( i.e. the offset to look into the Term Dictionary) .This automaton is used at query time to identify a starting point to look into the dictionary.When we run a query ( a **TermQuery** for example) :1) we give in input the query to the **In Memory Automaton**, an **Offset is returned**2) we access the location associated to the **Offset in the Term Dictionary**3) we **advance** to the **ByteRef** representation of the **TermQuery**4) if the term is a **match** for the **TermQuery** we return the **Posting List** associated

## 文档频率

This is simply the number of Documents in the corpus containing the term  t in the field f .For the term **game **we have **3 documents** in our corpus that contain the termin the field** title**

## 过帐清单

The posting list is the **sorted skip list** of DocIds that contains the related term.It’s used to return the documents for the searched term.Let’s have a deep look to a complete Posting List for the term **game** in the field **title**:

**0**:1:[2]:【6-10】，
**1** : 2 : [1，4]:【0-4，18-22】，
**2**:1:[1]:【0-4】

该过账列表的每个元素为:
*文档序号:词频:[词频数组]:[词频偏移量数组]。*

**文档序号** - >包含相关术语的语料库中的文档序号(Lucene ID)。

*例如*
根据每个发布列表元素的起始序号:
Doc0 **0** ，
Doc1 **1** ，
Doc2**2**
在**标题字段中包含术语**游戏**。**

**术语频率** - >术语在发布列表元素中出现的次数。
例如
**0**:1
**doc 0**包含 **1** 术语的出现**游戏**字段**标题**。
**1**:2
**Doc1**包含 **2** 术语的出现**游戏**字段**标题**。
**2**:1
**doc 2**包含 **1** 术语的出现**游戏**字段**标题**。

**术语位置数组** - >对于术语在发布列表元素中的每次出现，它包含字段内容中的相关位置。
例如
0:[2]
**doc 0**术语**游戏**在字段**标题**中的第一次出现在字段内容中的**第二个位置**。
( “标题”:“视频(1) **游戏(2)** 历史(3))
**1**:【1，4】
**Doc1**术语**游戏**在字段中第一次出现**标题**在字段内容中占据**第一位**。
**Doc1** 术语第二次出现**游戏**在字段**标题**在字段内容中占据**第四位**。
( 【标题】:**游戏(1)** 视频(2)回顾(3) **游戏(4)**)
**2**:【1】
**Doc0**术语第一次出现**游戏**领域**标题**占据
( 【标题】::**游戏(1)** 商店(2))

**术语偏移量数组** - >对于公告列表元素中术语的每一次出现，都包含字段内容中相关的字符偏移量。
例如
0:【6-10】
**doc 0**字段中第一次出现的术语**游戏****标题**从字段内容中的**第 6 个**字符开始，直到**第 10 个**字符(不包括)。
( "标题":"视频(1) **游戏(2)** …" )
( "标题":" 01234 5**6789**10**…"****)
**1**:【0-4，18-22】
..
**Doc1** 字段中第二次出现的术语**游戏**标题从字段内容中的**第 18 个**字符开始，直到**第 22 个**字符(不包括)。
( 【标题】:**游戏(1)** 视频(2)回顾(3) **游戏(4)**)
(【标题】: **0123** 4 视频(2)回顾(3)**18192021**22“【T83
( 【标题】:**游戏(1)** 商店(2))
(【标题】: **0123** 4 商店(2))**

 **## 实时文档

Live Documents is a lightweight data structure that keep the alive documents at the current status of the index.It simply associates a bit  1 ( alive) , 0 ( deleted) to a document.This is used to keep the queries aligned to the status of the Index, avoiding to return deleted documents.**Note** : **Deleted documents** in index data structures **are removed** ( and disk space released) only when a **segment merge happens**.This means that you can delete a set of documents, and the space in the index will be claimed back only after the first merge involving the related segments will happen.Assuming we delete the Doc2, this is how the Live Documents data structure looks like :

| 序数 | 活着的 |
| 0 | 1 |
| 1 | 1 |
| 2 | 0 |

**Ordinal** -> The Internal Lucene document ID
**Alive** -> A bit that specifies if the document is alive or deleted.

## 规范

Norms 是一种数据结构，它为每个文档的每个字段提供了长度标准化和提升因子。
数值与每个字段和每个文档相关联。
该值与内容值的**长度相关联，也可能与索引时间提升因子相关联。
在为查询检索的文档评分时使用规范。
具体来说，这是一种提高包含短字段术语的文档的相关性的方法。
步进时也可以关联一个提升因子。
**匹配启用了规范的查询时，短字段内容将胜过长字段内容**。**

| 田 | 标题 |
| 文档序号 | 标准 |
| 0 | 0.78 |
| 1 | 0.56 |
| 2 | 0.98 |

## 模式配置

When configuring a field in Solr ( or directly in Lucene) it is possible to specify a set of field attributes to control which data structures are going to be produced .Let’s take a look to the different Lucene Index Options

| Lucene 索引选项 | Solr 模式 | 描述 | 当…时使用 |
| 没有人 | indexed="false " | 不会建立倒排索引。 | 你不需要在你的文档库中搜索。 |
| 文件（documents 的简写） | omitTermFreqAnd
Positions = " true " | 每个术语的发布列表只包含文档 id(序号),没有其他内容。
  例如
游戏- > **0** ， **1** ， **2** | –你不需要使用短语或位置查询在语料库中搜索。
-您不需要分数受到文档字段中术语出现次数的影响。 |
| 文档和频率 | omitPositions="true " | 每个术语的发布列表将只包含文档中的文档 id(序号)和术语频率。
  例如
游戏->**0**:1，**1**:2， **2** : 1 | ——你 不需要 在你的语料库 中用短语或者位置 查询搜索。
–您确实需要评分来考虑术语频率 |
| DOCS _ AND _ freq _ AND _
职位 | indexed="true "时的默认值 | 每个术语的发布列表还将包含术语位置。
  例如
**0**:1:[2]， **1** : 2 : [1，4]，**2**:1:[ | –你 确实需要 在你的语料库 中用短语或位置 查询进行搜索。
–您确实需要评分来考虑术语频率 |
| 文档 _ 和 _ 频率 _ 和 _
位置 _ 和 _ 偏移 | storeOffsetsWithPositions = " true " | The posting list for each term will contain the term offsets in addition.例如:game->**0**:1:[2]:【6-10】，
**1**:2:[1，4]:【0-4，18-22】，
**2**:1:[1]:【0-4】 | –您想要使用发布荧光笔。
一种快速版本的高亮显示，使用发布列表而不是术语向量。 |
| omitNorms | omitNorms="true " | 不会建立规范数据结构 | –不需要提升短字段内容
–不需要提升每个字段的索引时间 |

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929232332/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929232332/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们也提供关于这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929232332/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Lucene 倒排索引的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**