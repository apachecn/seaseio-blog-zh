# Apache Solr 神经搜索

> 原文：<https://web.archive.org/web/sease.io/2022/01/apache-solr-neural-search.html>

在 **Apache Solr** 中神经搜索的第一个里程碑已经由 **Sease** [ [1](https://web.archive.org/web/20220929231903/https://github.com/apache/solr/commit/c309a8a65d8868e8ae034f80d5682b1587db3863) ]通过**Alessandro Benedetti**(Apache Lucene/Solr PMC 成员和提交者)和**Elia por ciani**(Sease R&D 软件工程师)的工作贡献给了开源社区。
它依赖于 **Apache Lucene** 实现[【2】](https://web.archive.org/web/20220929231903/https://github.com/apache/lucene-solr/commit/b36b4af22bb76dc42b466b818b417bcbc0deb006)进行 K 近邻搜索。
特别感谢 Christine Poerschke、Cassandra Targett、 Michael Gibney、 以及所有其他在文章最后阶段给予帮助的审稿人。即使是一条评论也很受赞赏，如果我们取得了进步，这总是要感谢社区。

让我们从一个简短的介绍开始，关于神经方法如何改进搜索。

我们可以将搜索总结为四个主要领域:

*   **生成指定信息需求的查询**的表示
*   **生成文档**的表示，该表示捕获了所包含的信息
*   **匹配查询和来自信息语料库的文档**表示
*   **为每个匹配的文档分配一个分数**，以便根据结果中的相关性建立一个有意义的文档排名

神经搜索是从神经信息检索[【3】](https://web.archive.org/web/20220929231903/https://www.microsoft.com/en-us/research/uploads/prod/2017/06/fntir2018-neuralir-mitra.pdf)学术领域衍生出来的一个行业，它专注于用基于神经网络的技术来改进这些领域中的任何一个。

## 人工智能、深度学习和向量表示

我们越来越频繁地听到人工智能(AI 从此)如何渗透到我们生活的许多方面。

当我们谈论人工智能时，我们指的是一套技术，使机器能够学习并显示类似人类的智能。

随着近年来计算机能力的强劲和稳定发展，人工智能已经出现了复苏，现在它被用于许多领域，包括软件工程和信息检索(管理搜索引擎和类似系统的科学)。

特别是，深度学习[【4】](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Deep_learning)的出现引入了深度神经网络的使用，以解决经典算法非常具有挑战性的复杂问题。

为了这篇博文，知道深度学习可以用来产生信息语料库中查询和文档的向量表示就足够了。

### 密集向量表示

传统的[倒排索引](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Inverted_index)可以被认为是将文本建模为“稀疏”向量，其中语料库中的每个术语对应于一个向量维度。在这样的模型中(参见[单词袋](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Bag-of-words_model)方法)，维度的数量与术语字典基数相对应，并且任何给定文档的向量大多包含零(因此它被称为稀疏的，因为在整个字典中只有少数术语会出现在任何给定的文档中)。

密集向量表示与基于术语的稀疏向量表示形成对比，因为它将近似的语义提取到固定(和有限)数量的维度中。

这种方法中的维数通常比稀疏情况低得多，并且任何给定文档的向量都是密集的，因为其大部分维数由非零值填充。

与稀疏方法(使用标记化器直接从文本输入生成稀疏向量)相反，生成向量的任务必须在 Apache Solr 外部的应用程序逻辑中处理。

各种深度学习模型，如 BERT[【5】](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/BERT_(language_model))能够将文本信息编码为密集向量，用于密集检索策略。

更多信息，你可以参考我们的的这篇[博客。](https://web.archive.org/web/20220929231903/https://sease.io/2021/12/using-bert-to-improve-search-relevance.html)

## 近似最近邻

给定一个对信息需求建模的密集向量`v`,提供密集向量检索的最简单方法是计算距离(欧几里德距离、点积距离等)。)和代表信息语料库中的文档的每个向量`d`之间。

这种方法相当昂贵，因此目前正在积极研究许多近似策略。

近似最近邻搜索算法返回结果，其与查询向量的距离至多是查询向量与其最近向量的距离的![c](img/11026e8a99bd2f53d8b0d15c25a34ad7.png)倍。
这种方法的好处是，在大多数情况下，近似的最近邻几乎与精确的最近邻一样好。
特别是，如果距离度量准确地捕捉到了用户质量的概念，那么距离上的小差异应该无关紧要[【6】](https://web.archive.org/web/20220929231903/https://ieeexplore.ieee.org/document/4031381)

## 分层可导航小世界图

在 Apache Lucene 中实现并由 Apache Solr 使用的策略基于可导航的小世界图。

为高维向量[【7】](https://web.archive.org/web/20220929231903/https://doi.org/10.1016/j.is.2013.10.006)[【8】](https://web.archive.org/web/20220929231903/https://arxiv.org/abs/1603.09320)[【9】](https://web.archive.org/web/20220929231903/https://erikbern.com/2018/06/17/new-approximate-nearest-neighbor-benchmarks.html)[【10】](https://web.archive.org/web/20220929231903/https://www.benfrederickson.com/approximate-nearest-neighbours-for-recommender-systems/)提供高效的近似最近邻搜索。

分层可导航小世界图(HNSW)是一种基于 ***邻近邻居图*** :
概念的方法，与信息语料库相关联的向量空间中的每个向量唯一地与一个

图形![G(V,E)](img/9e463cb8c6dd1af9eaa12b92e2650385.png)中的顶点![v_{i}\in V](img/c229333b910bb2bdfc9f364f4530e11a.png)。

顶点通过基于它们的接近度的边连接，越接近的顶点(根据距离函数)被连接。

构建图表受超参数的影响，这些超参数规定了每层要构建多少个连接以及要构建多少个层。

在查询时，导航邻居结构以找到离目标最近的向量，从种子节点开始，随着我们越来越接近目标而迭代。

我发现[这个博客](https://web.archive.org/web/20220929231903/https://www.pinecone.io/learn/hnsw/)对于深入探讨这个话题非常有用。

## Apache Lucene 实现

首先要注意的是，当前的 Lucene 实现不是分层的。
所以图中只有一个单层，见原《吉拉追踪发展进度》一期中的最新评论[【11】](https://web.archive.org/web/20220929231903/https://issues.apache.org/jira/browse/LUCENE-9004?focusedCommentId=17393216&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-17393216)。
主要原因是为了在 Apache Lucene 生态系统中找到更简单的设计、开发和集成过程。
大家一致认为，引入层次化的分层结构将在低维向量管理和查询时间方面带来好处(减少要遍历的候选节点)。
该实施正在进行中[【12】](https://web.archive.org/web/20220929231903/https://issues.apache.org/jira/browse/LUCENE-10054)。

那么，Apache Lucene 的可导航小世界图和 K 近邻功能涉及哪些组件呢？
让我们来探索代码:

如果你对 Lucene 内部和编解码器不感兴趣，你可以跳过这一段

***org . Apache . Lucene . document . knnvectorfield***是入口点:
它在索引时需要**向量维度**和**相似度函数**(构建 NSW 图时使用的向量距离函数)。
通过**# setVectorDimensionsAndSimilarityFunction**方法在**org . Apache . Lucene . document . field type**中设置。

更新文档字段 schema***org . Apache . Lucene . index . indexing chain # updateDocFieldSchema***时，从 **FieldType** 中提取信息并保存在***org . Apache . Lucene . index . indexing chain . field schema***中。

并从***field schema******KnnVectorField***配置最终到达**org . Apache . Lucene . index . field info**中的***org . Apache . Lucene . index . indexing chain # initializefield info***。

现在 Lucene 编解码器拥有了构建 NSW 图所需的所有特定于字段的配置。让我们来看看如何:

首先从***org . Apache . Lucene . codecs . Lucene 90 . Lucene 90 codec # Lucene 90 codec***你会看到 KNN 向量的默认格式是***org . Apache . Lucene . codecs . Lucene 90 . hnswvectorsformat***。

关联的索引编写器是******org . Apache . Lucene . codecs . Lucene 90 . Lucene 90 hnswvectorwriter******。
当将字段写入***org . Apache . Lucene . codecs . Lucene 90 . Lucene 90 hnswvectorwriter # write field***中的索引时，该组件可以访问之前初始化的 ***FieldInfo*** 。
在写一个***KnnVectorField***的时候，会涉及到***org . Apache . Lucene . util . hnsw . hnsw graph builder***，最后构建一个
***org . Apache . Lucene . util . hnsw . hnsw graph***。

## Apache Solr 实现

> 可从 Apache Solr 9.0 获得
> 
> 预计 2022 年第一季度

这第一个贡献允许索引单值密集矢量场，并使用近似距离函数搜索 K-最近邻。

当前功能:

*   DenseVectorField 类型
*   Knn 查询解析器

### DenseVectorField

密集向量字段提供了索引和搜索浮动元素的密集向量的可能性。

例如

`[1.0, 2.5, 3.7, 4.1]`

下面是应该如何在模式中配置`DenseVectorField`:

```
**<fieldType** name="knn_vector" class="solr.DenseVectorField" vectorDimension="4" similarityFunction="cosine"**/>**
**<field** name="vector" type="knn_vector" indexed="true" stored="true"**/>**
```

| ****参数名称**** | **必需的** | **默认** | **描述** | **接受值** |
| `vectorDimension` | 真实的 |   | 要传入的密集向量的维度。 | 整数< = 1024 |
| 【 | False | 【 | Vector similarity function; used in search to return top K most similar vectors to a target vector. | 【 ,  【  or  【 . |

*   `euclidean` : [欧几里德距离](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Euclidean_distance)
*   `dot_product` : [点积](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Dot_product)。**注**:该相似度旨在作为执行余弦相似度的优化方式。为了使用它，所有向量都必须是单位长度的，包括文档和查询向量。对非单位长度的向量使用点积会导致错误或较差的搜索结果..
*   `cosine` : [余弦相似度](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Cosine_similarity)。**注**:执行余弦相似度的首选方法是将所有向量归一化为单位长度，而不是使用点积。只有在需要保留原始向量并且无法提前对其进行归一化时，才应该使用该函数。

DenseVectorField 支持属性:`indexed`、`stored`。

**注意**目前不支持多值

#### 自定义索引编解码器

要使用以下自定义编解码器格式的高级参数和 HNSW 算法的超级参数，请确保在 solrconfig.xml 中设置此配置:

```
 **<codecFactory** class="solr.SchemaCodecFactory"**/>**
...
```

以下是如何使用高级编解码器超参数配置`DenseVectorField`:

```
**<fieldType** name="knn_vector" class="solr.DenseVectorField" vectorDimension="4" similarityFunction="cosine" codecFormat="Lucene90HnswVectorsFormat" hnswMaxConnections="10" hnswBeamWidth="40"**/>**
**<field** name="vector" type="knn_vector" indexed="true" stored="true"**/>**
```

| **参数名称** | **必需的** | **默认** | **描述** | **接受值** |
| `codecFormat` | 错误的 | `Lucene90HnswVectorsFormat` | 指定要使用的 knn 编解码器实现 | `Lucene90HnswVectorsFormat` |
| ``hnswMaxConnections`` | 错误的 | `16` | **`Lucene90HnswVectorsFormat` only:**
控制有多少个最近邻候选连接到新节点。
与纸上的`**M**`含义相同[【8】](https://web.archive.org/web/20220929231903/https://arxiv.org/abs/1603.09320)。 | `Integer` |
| `hnswBeamWidth` | 错误的 | `100` | **`Lucene90HnswVectorsFormat` only:** 它是在图中搜索每个新插入的节点时要跟踪的最近邻候选的数量。
与纸上的`**efConstruction**`含义相同[【8】](https://web.archive.org/web/20220929231903/https://arxiv.org/abs/1603.09320)。 | `Integer` |

请注意，`codecFormat`接受值在未来版本中可能会改变。

| **注** | 只有默认编解码器支持 Lucene 索引向后兼容性。如果您选择在您的模式中定制`codecFormat`，升级到 Solr 的未来版本可能需要您在升级之前切换回默认编解码器并优化您的索引以将其重写为默认编解码器，或者在升级之后从头开始重新构建您的整个索引。 |

关于 HNSW 实施的超参数，详见[【8】](https://web.archive.org/web/20220929231903/https://arxiv.org/abs/1603.09320)。

### 如何索引向量

下面是一个`DenseVectorField`应该如何被索引:

#### **JSON**

```
[{ "id": "1",
"vector": [1.0, 2.5, 3.7, 4.1]
},
{ "id": "2",
"vector": [1.5, 5.5, 6.7, 65.1]
}
]
```

#### **XML**

```
 **<field** name="id"**>**1
**<field** name="vector"**>**1.0
**<field** name="vector"**>**2.5
**<field** name="vector"**>**3.7
**<field** name="vector"**>**4.1

**<field** name="id"**>**2
**<field** name="vector"**>**1.5
**<field** name="vector"**>**5.5
**<field** name="vector"**>**6.7
**<field** name="vector"**>**65.1 
```

#### Java-**SolrJ**

```
**final** SolrClient client = getSolrClient();

**final** SolrInputDocument d1 = **new** SolrInputDocument();
d1.setField("id", "1");
d1.setField("vector", **Arrays**.asList(1.0f, 2.5f, 3.7f, 4.1f));

**final** SolrInputDocument d2 = **new** SolrInputDocument();
d2.setField("id", "2");
d2.setField("vector", **Arrays**.asList(1.5f, 5.5f, 6.7f, 65.1f));

client.add(**Arrays**.asList(d1, d2));
```

## knn 查询解析器

`knn` K-Nearest Neighbors 查询解析器允许根据给定字段中的索引密集向量，找到与目标向量最近的 K 个文档。

它采用以下参数:

| **参数**名称 | **必需的** | **默认** | **描述** |
| `f` | 真实的 |   | 要搜索的 DenseVectorField。 |
| `topK` | 错误的 | 10 | 要返回多少个 k-nearest 结果。 |

以下是如何运行 KNN 搜索:

```
&q={!knn f=vector topK=10}[1.0, 2.0, 3.0, 4.0]
```

检索到的搜索结果是与输入`[1.0, 2.0, 3.0, 4.0]`中的向量最接近的 K 个，由索引时配置的 similarityFunction 排序。

### 过滤查询的用法

`knn`查询解析器可以用于过滤查询:

```
&q=id:(1 2 3)&fq={!knn f=vector topK=10}[1.0, 2.0, 3.0, 4.0]
```

`knn`查询解析器可以用于过滤查询:

```
&q={!knn f=vector topK=10}[1.0, 2.0, 3.0, 4.0]&fq=id:(1 2 3)
```

| **重要的** | 当在这些场景中使用`knn`时，确保您清楚地了解过滤查询在 Apache Solr 中是如何工作的:
从主查询`q`得到的文档 id 的排序列表与从每个过滤查询`fq`得到的文档 id 的集合相交。例如，从`q` = `[ID1, ID4, ID2, ID10]`得到的排序列表从`fq` = `{ID3, ID2, ID9, ID4}` = `[ID4,ID2]`得到 |

### 用作重新排序查询

`knn`查询解析器可用于对第一轮查询结果进行重新排序:

```
&q=id:(3 4 9 2)&rq={!rerank reRankQuery=$rqq reRankDocs=4 reRankWeight=1}&rqq={!knn f=vector topK=10}[1.0, 2.0, 3.0, 4.0]
```

| **重要的** | 在重新分级中使用`knn`时，注意`topK`参数。
仅当来自第一遍的文档`d`在要搜索的目标向量的 K 个最近邻居(**在整个索引**中)内时，才计算第二遍得分(从 knn 得出)。
这意味着无论如何都要对整个索引执行第二遍`knn`，这是当前的限制。
结果的最终排序列表将第一次通过分数(主查询`q`)加到第二次通过分数(到要搜索的目标向量的近似相似性函数距离)上，再乘以一个乘法因子(重排序权重)。
因此，如果文档`d`没有出现在 knn 结果中，即使目标查询向量的距离向量计算不为零，您也可以获得对原始分数的零贡献。关于使用 ReRank 查询解析器的细节可以在 Apache Solr Wiki[【13】](https://web.archive.org/web/20220929231903/https://solr.apache.org/guide/query-re-ranking.html)部分找到。 |

## 未来作品

这只是 Apache Solr 神经功能的开始，更多功能即将推出！
我希望你喜欢这个概述，并继续关注更新！

## 参考

[6] *安多尼，a；因迪克，P. (2006 年 10 月 1 日)。“高维近似最近邻的近似最优哈希算法”。 *2006 年第 47 届 IEEE 计算机科学基础年会(FOCS 06)*。第 459-468 页。[CiteSeerX](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/CiteSeerX_(identifier))10 . 1 . 1 . 142 . 3471。[doi](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Doi_(identifier)):[10.1109/focs . 2006.49](https://web.archive.org/web/20220929231903/https://doi.org/10.1109%2FFOCS.2006.49)。[ISBN](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/ISBN_(identifier))[978-0-7695-2720-8](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/Special:BookSources/978-0-7695-2720-8)。*

[7]尤里·马尔科夫，亚历山大·波诺马伦科，安德烈·洛格维诺夫，弗拉基米尔·克雷洛夫，
基于可导航小世界图的近似最近邻算法，
信息系统，
第 45 卷，
2014，
第 61-68 页，
ISSN 0306-4379，
https://doi.org/10.1016/j.is.2013.10.006.

[8]马尔科夫，尤里；德米特里·亚舒宁(2016)。“使用分层可导航小世界图的高效和健壮的近似最近邻搜索”。[arXiv](https://web.archive.org/web/20220929231903/https://en.wikipedia.org/wiki/ArXiv_(identifier)):[1603.09320](https://web.archive.org/web/20220929231903/https://arxiv.org/abs/1603.09320)[[cs。DS](https://web.archive.org/web/20220929231903/https://arxiv.org/archive/cs.DS) 。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们在搜索培训中做[人工智能](https://web.archive.org/web/20220929231903/https://sease.io/artificial-intelligence-in-search-training)和针对搜索的关于深度学习的特定[一天培训？
我们也提供这些话题的咨询，](https://web.archive.org/web/20220929231903/https://sease.io/information-retrieval-mini-training)[如果你想用人工智能的力量把你的搜索引擎提升到一个新的水平，请联系](https://web.archive.org/web/20220929231903/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于 Apache Solr 神经搜索的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！