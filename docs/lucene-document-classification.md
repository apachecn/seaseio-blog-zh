# Lucene 文档分类

> 原文：<https://web.archive.org/web/sease.io/2015/07/lucene-document-classification.html>

## 介绍

这篇博文描述了 Lucene 分类模块中使用的使文本分类适应文档(多字段)分类的方法。

机器学习和搜索一直是紧密联系在一起的。
机器学习可以在很多方面帮助改善搜索体验，从文档语料库中提取更多信息，自动分类并聚类。
另一方面，搜索相关的数据结构在内容上与用来解决问题的 ML 模型非常相似，所以我们可以用它们来构建自己的算法。
但是让我们按顺序来……
在这篇文章中，我们将探讨自动分类对于简化用户搜索体验有多重要，以及如何能够直接从 Lucene、从我们已经存在的索引、没有任何外部依赖或训练模型中获得一个简单的、开箱即用的无痛分类。

## 分类

来自维基百科:

**机器学习中的****统计****分类** 的问题是识别到一组类别(子群体)中的一个新的 **观察** 所属，在此基础上建立一个 **训练集一个例子是将给定的电子邮件分配到****垃圾邮件**或**非垃圾邮件**类别中，或者根据观察到的患者特征(性别、血压、某些症状的存在与否等)将诊断分配给给给定的患者。).

分类是一个众所周知的问题，每天都很容易验证分类器的效用。
在与搜索引擎打交道时，我们的文档有一个类别(一般由人类指定)是如此常见。能够自动提取缺少类别的文档的类别不是很有用吗？根据经验自动对文档进行分类，这难道不是一件有趣的事情吗？(这意味着使用我们的搜索系统中已经分类的所有文档)
如果没有任何外部工具，没有任何额外的昂贵的模型培训活动，这样做难道不会令人惊讶吗？
我们的搜索系统里已经有很多信息了，让我们利用起来吧！

## Lucene 分类模块

为了能够提供分类能力，我们的系统通常需要一个经过训练的模型。
分类是由监督机器学习算法解决的问题，这意味着人类需要提供一个训练集。
分类训练集是一组文档，手动标记有正确的类别。
这是分类系统将用于对即将到来的未见过的文档进行分类的“经验”。
建立一个经过训练的模型是昂贵的，并且需要存储在特定的数据结构中。
当我们的索引中已经有数百万的文档，已经被人类分类并准备好被搜索时，真的有必要建立一个外部训练的模型吗？Lucene 索引已经有了非常有趣的数据结构，它将术语与文档(倒排索引)、字段与术语(术语向量)以及我们需要的关于术语频率和文档频率的所有信息联系起来。
在没有任何外部工具或数据结构的情况下，在我们的搜索引擎内，一个已经标记了特定领域的适当类别的文档语料库可以成为构建我们的分类**的真正有用的资源。**
基于这些观察，Apache Lucene开源项目**4.2 版本引入了一个文档分类** 模块来管理使用索引数据结构的文本分类。
这个模块的外观是文本分类器，一个简单的接口(有 3 个可用的实现):

公共接口分类器{ /** *给定的文本字符串分配一个类(带分数*
* @param text 一个包含要分类的文本的字符串*
@ return a { @ link classification result }保存分配的 T 类型的类和分数*

ClassificationResult assign class(字符串文本)引发 IOException

/** *获取分配给给定文本字符串的所有类(按分数降序排列)。
@param text 包含待分类文本的字符串*
@返回{@link ClassificationResult}的完整列表，包括类别和分数。如果分类器不能生成列表，则返回 null。*/

List getClasses(字符串文本)引发 IOException

/** *获取分配给给定文本字符串的第一个 max 类(按分数降序排列)。
@param text 包含待分类文本的字符串*
@param max 返回列表元素的个数*
@返回{@link ClassificationResult}的完整列表，包括类别和分数。切割“最大”数量的元素。如果分类器不能生成列表，则返回 null。*/

List getClasses(String text，int max)抛出 IOException
}

可用的实现有:

*   **knarestneighborclassifier**–基于 MoreLikeThis 的 k 近邻分类器[【2】](https://web.archive.org/web/20220929232419/https://en.wikipedia.org/wiki/K-nearest_neighbors))

*   【based NaiveBayes 分类器——一个基于 Lucene 的简单 NaiveBayes 分类器[【3】](https://web.archive.org/web/20220929232419/https://en.wikipedia.org/wiki/Naive_Bayes_classifier)

*   **booleanpectronclassifier**–基于感知器[【4】](https://web.archive.org/web/20220929232419/https://en.wikipedia.org/wiki/Perceptron)的布尔分类器。使用 TermsEnum.totalTermFreq 在每个字段和每个文档的基础上计算权重，然后将相应的 FST 用于类别分配。

让我们详细看看前两个:

###### KNearestNeighborClassifier

This classifier is based on the Lucene More like This [[5]](https://web.archive.org/web/20220929232419/https://solr.apache.org/guide/6_6/morelikethis.html). A feature able to retrieve similar documents to a seed one ( or a seed text) calculating the similarity between the docs on a field base.It takes in input the list of fields to take in consideration ( with relative boost factor to increase the importance of some field over the others).The idea behind this algorithm is simple : – given a set of relevant fields for our classification – given a field containing the class of the documentIt retrieves all the similar documents to the Text in input using the MLT.Only the documents with the class field valorised, are taken in consideration.The **top k documents** in result are evaluated, and the class extracted from all of them.Then a ranking of the retrieved classes is made, based on the frequency of the class in the top K documents.Currently the algorithm takes in consideration only the frequency of the class to calculate its score.One limitation is that is not taking in consideration the ranking of the class.This means that if in the top K, we have the first k/2 documents of class C1 and the second k/2 document of class C2, both the classes will have the same score (LUCENE-6654) [[6]](https://web.archive.org/web/20220929232419/https://issues.apache.org/jira/browse/LUCENE-6654).This classifier works on top of the Index, let’s quickly have an overview of the constructor parameters :  

| 参数 | 描述 |
| 叶片阅读器 | 将用于读取索引数据结构和对文档进行分类的索引读取器 |
| 分析者 | 用于分析输入的不可见文本的分析器 |
| 询问 | 应用于索引文档的过滤器。只有满足查询的那些将被用于分类 |
| k | 在 MLT 结果中选择的用于查找最近邻的顶级文档的数量 |
| minDocsFreq | 只有当一个术语(来自输入文本)至少出现在索引中的这个最小数量的文档中时，算法才会考虑该术语 |
| minTermFreq | (来自输入文本的)术语只有在输入中出现至少这个最小次数时，算法才会考虑 |
| 类别字段名称 | 包含文档类别的字段。它必须出现在索引文档中。**必须存储** |
| 文本字段名称 | 进行分类时要考虑的字段列表。它们必须出现在索引文档中 |

**注意** : `MoreLikeThis` 基于文档中的术语构造 Lucene 查询。这是通过从已定义的字段列表中提取术语来实现的(下面的文本字段
名称参数)。
你通常会读到，为了获得最佳结果，字段应该存储术语向量**。**
**在我们的例子中，文本要分门别类是看不见的，它是不加索引的文档。**
**所以根本不用术语 Vector。**

###### SimpleNaiveBayesClassifier

This Classifier is based on a simplistic implementation of a Naive Bayes Classifier [[3]](https://web.archive.org/web/20220929232419/https://en.wikipedia.org/wiki/Naive_Bayes_classifier).It uses the Index to get the Term frequencies of the terms, the Doc frequencies and the unique terms.First of all it extract all the possible classes, this is obtained getting all the terms for the class field.Then , each class is scored based on– how frequent is the class in all the classified documents in the index– how much likely the tokenized text is to belong to the classThe details of the algorithm go beyond the scope of this blog post.This classifier works on top of the Index, let’s quickly have an overview of the constructor parameters :

| 参数 | 描述 |
| 叶片阅读器 | 将用于读取索引数据结构和对文档进行分类的索引读取器 |
| 分析者 | 用于分析输入的不可见文本的分析器 |
| 询问 | 应用于索引文档的过滤器。只有满足查询的那些将被用于分类 |
| 类别字段名称 | 包含文档类别的字段。它必须出现在索引文档中 |
| 文本字段名称 | 进行分类时要考虑的字段列表。它们必须出现在索引文档中 |

**注** : 朴素贝叶斯 分类器 作用于索引中的**术语**。这意味着它从索引中提取类字段的标记。**每个令牌将被视为类别**，并将看到相关的分数。

这意味着你必须在为 classField 选择的分析中小心谨慎，最好使用一个包含该类的 not tokenizer 字段(如有必要，一个 copy field)

## 文档分类扩展

The original Text Classifiers are perfectly fine, but what about Document classification ?In a lot of cases the information we want to classify is actually composed of a set of fields with one or more value each one.Each field content contributes to the classification ( with a different weight to be precise).Given a simple News article, the title is much more important for the classification in comparison with the text and the author.But even if not so relevant even the author can have a little part in the class assignation.Lucene atomic unit of information is the **Document. **

文档是索引和搜索的单位。文档是一组字段。每个字段都有一个名称和一个文本值。

这种数据结构是新一代分类器的完美输入，新一代分类器将受益于增强信息来分配相关类别。
每当一个简单的文本有歧义时，包括其他字段在内的分析就大大提高了分类的精度。
详细内容:

*   *   输入文档中的**字段**内容将与索引中的数据结构进行比较，仅与该**字段** 相关
    *   输入字段将根据其自身的**分析链**进行分析，从而实现更大的灵活性和精确性
    *   一个**输入字段可以被放大**，以影响更多的分类。这样，输入文档的不同部分将具有不同的相关性来发现文档的类别。

A new interface is provided :

公共接口文档分类器< T > 

/* **给给定的分配一个类(带分数){@ linkorg . Apache . Lucene . document }**@ paramdocumenta {@ linkorg . Apache 字段被视为分类的要素。*@ returna {@ linkorg . Apache . Lucene . classification . result }保存分配的类的类型T 和分数*/

classification result<T>assign class(Document Document)抛出io exception；

/** *获取分配给 given 的所有类(按分数降序排列){@ linkorg . Apache . Lucene . document }。**@ param文档a {@ linkorg . Apache . Lucene . document }待分类。字段被视为分类的要素。*@ return完整列表{@ linkorg . Apache . Lucene . classification . result }、班级和分数。如果分类器不能列表，则返回空值 。/

列表<分类结果<T>>get classes(Document Document)抛出io exception；

/** *将第一个 max 类(按分数降序排序)赋给给定的文本字符串。**@ param文档a {@ linkorg . Apache . Lucene . document }待分类。字段被视为分类的要素。*@ parammax返回列表元素的个数*@ return{@ linkorg . Apache . Lucene . classification . result }的整个列表，类和分数。切割“最大”数量的元素。如果分类器不能列表，则返回 null 。 */

列表<分类结果<T>>get classes(Document Document， int max) 抛出io exception；

扩展了 2 个分类器以提供新功能:

*   **knarestneighbordocumentclassifier**
*   **SimpleNaiveBayesDocumentClassifier**

第一个实现可作为本期《吉拉》的投稿:

*[LUCENE-6631](https://web.archive.org/web/20220929232419/https://issues.apache.org/jira/browse/LUCENE-6631)*

现在有了一个新的接口，将它与 Apache Solr 集成会容易得多。

请继续关注 Solr 分类集成。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929232419/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929232419/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929232419/https://sease.io/contacts)！

// stay always up to date

## 订阅我们的时事通讯

你喜欢这个关于 Lucene 文档分类的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！