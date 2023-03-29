# 离线搜索质量评估:分级评价员(RRE)

> 原文：<https://web.archive.org/web/sease.io/2021/01/offline-search-quality-evaluation-rated-ranking-evaluator-rre.html>

## 介绍

随着[Rated Ranking Evaluator Enterprise](https://web.archive.org/web/20220929231153/https://sease.io/2019/12/road-to-rated-ranking-evaluator-enterprise.html)即将推出，我们借此机会详细解释为什么离线搜索质量评估在当今如此重要，以及您已经可以使用 Rated Ranking Evaluator 开源库做些什么。
随着 V1 发布日期的临近，更多的消息将会很快到来。
敬请期待！

## 搜索质量评估

评估是每一项科学发展的基础。
科学家提出假设来模拟现实世界的现象，并通过将它们的输出与自然界的观察结果进行比较来验证它们。

评价在信息检索领域起着非常关键的作用。
研究人员和实践者开发排名模型来解释用户表达的信息需求**(查询)**和包含在可用资源**(语料库)**中的信息**(搜索结果)**之间的关系，并通过将它们的结果与一组观察结果**(隐式/显式用户反馈)**进行比较来测试这些模型。

搜索质量有多种解释，但是，本博客只关注其中一种:搜索系统找到与用户相关的信息的有效性**(搜索相关性)**。

有两种类型的观察用于评估目的:
**(a)显式反馈**(相关性注释)，以及 **(b)隐式反馈**(可观察的用户交互)。

大多数情况下，对信息检索系统的评估是在离线和/或在线的情况下进行的，一个或多个利益相关者运行一组不可重复的查询，并根据当时的感觉(以后可能会改变)评估搜索结果。

有必要在这个过程中引入科学的结构:

*   *   收集基本事实观察**(来自隐式/显式反馈的评级)**
    *   针对搜索系统运行有限且明确定义的相应查询列表
    *   将结果与**基本事实**进行比较，探索不同的**搜索质量指标**

这必须是**可再现的、**持久的和**可解释的**。

下面是**评级评估员**(以下简称 **RRE** )基于 **Apache Lucene** 的搜索引擎 **Apache Solr** 和 **Elasticsearch** 的**离线搜索质量评估库**

## 这是什么？

[Rated rank Evaluator(RRE)](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator)是一个搜索质量评估库，用于评估来自搜索系统的结果的质量。

它帮助搜索工程师和商业利益相关者:

*   *   您是否正在调整/实施/更改/配置搜索基础架构？
    *   你想要从你最近的改变中得到改进的证据吗？

RRE 会帮你的。

RRE 在“技术”层面上正式确定了搜索系统满足用户信息需求的程度，结合了表达基本事实评级的可能性和用几个评估指标评估搜索系统质量。
RRE 提供人类可读的输出，对非技术利益相关者也很有用。

在搜索系统的开发和发展过程中，它鼓励一种**增量** / **迭代** / **可重复**的方法:假设我们从 x.y 版本开始我们的系统:当需要对它的配置应用一些相关的更改时，新版本被单独评估(源代码控制在这里有助于跟踪发展的配置文件)。

RRE 对 input 中的所有搜索系统版本执行评估过程，并提供它们之间的差值/趋势，因此您可以立即发现您感兴趣的指标中的改进或退步。

## 投入

为了执行评估，RRE 需要以下东西:

*   *   一个或多个 [**语料库** / **测试集**](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/What%20We%20Need%20To%20Provide#corpus) :这些是特定领域的代表性数据集，将用于填充和查询目标搜索平台
    *   一个或多个 [**配置集**](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/What%20We%20Need%20To%20Provide#configuration-sets) :管理目标搜索引擎的 Apache Solr/Elasticsearch 配置。
    *   一个或多个 [**评级集**](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/What%20We%20Need%20To%20Provide#ratings) :地面真相、<查询列表、关联到相关性评级的文档>对

## 基本事实定义(评级)

首先，您需要定义您的基本事实:
**<查询，标记有**相关性标签**的文档>** 对，该标签说明文档与给定查询的相关性。

评级文件以 JSON 格式提供，是 RRE 的核心输入。
每个评级文件都是一组结构化的 **<查询、文档>** 对(即给定查询的相关文档)。

在收视率文件中，我们可以定义 RRE 支持的各方面的信息需求:**语料库**，**主题**，**查询组**，以及**查询**。

当前实施使用可配置的判断范围:

例如

*   *   1 = >勉强相关
    *   2 = >相关
    *   3 = >非常相关

在“relevant_documents”节点中，您可以通过以下方式之一提供判断:

```
"relevant_documents": {
   "docid1": {
       "gain": 2
   }, 
   "docid2": {
       "gain": 2
   }, 
   "docid3": {
       "gain": 3
   }, 
   "docid5": {
       "gain": 2
   }, 
   "docid99": {
       "gain": 3
   }
}
```

```
"relevant_documents": {
   "2": ["docid1", "docid2", "docid5"],
   "3": ["docid3", "docid99"]   
}
```

请注意，如果某个文档与给定的查询不相关，它不会出现在相关列表中。换句话说，没有“0”判断值。

评级文件的更完整示例:

```
{
  "index": "<string>",
  "corpora_field": "<string>",
  "id_field": "<string>",
  "topics": [
    {
      "description": "<string>",
      "query_groups": [
        {
          "name": "<string>",
          "queries": [
            {
              "template": "<string>",
              "placeholders": {
                "$key": "<value>",
              }
            }
          ],
          "relevant_documents": [
            {
              "document_id": {
                "gain": "<number>"
              }
            }
          ]
        }
      ]
    }
  ],
  "query_groups": [
    {
      "name": "<string>",
      "queries": [
        {
          "template": "<string>",
          "placeholders": {
            "$key": "<value>",
          }
        }
      ],
      "relevant_documents": [
        {
          "document_id": {
            "gain": "<number>"
          }
        }
      ]
    }
  ],
  "queries": [
    {
      "template": "<string>",
      "placeholders": {
        "$key": "<value>",
      }
    }
  ]
}
```

*   *   `index`:弹性搜索中的索引名 Solr 中的集合名
    *   `corpora_field`:外部 Solr/ElasticSearch 不需要，对应于语料库文件名
    *   `id_field`:模式中代表文档 ID 的字段
    *   `topics`:主题和/或查询组的可选列表
        *   `description`:报告输出中使用的主题标题
        *   `query_groups`:以相同名称和主题分组的查询列表
            *   `name`:用于报告的查询名称
            *   `queries`:针对主题执行的查询列表
                *   `template`:该查询使用的模板的字符串名称
                *   占位符:模板中要替换的键值对的对象
        *   `relevant_documents`:列出带有映射文档的对象，以获得或关联值
    *   `query_groups`:带有相关查询的对象列表。可以存在于`topics`之外，是可选的
    *   `queries`:必选*用于评估的模板和占位符替换的对象列表。
        *   *如果`topics`和`query_groups`不存在

## 查询模板

对于每个查询(或每个查询组)，可以定义一个查询模板，它是包含一个或多个占位符的查询的定义。然后，在评级文件中，您可以引用其中一个已定义的模板，并为每个占位符提供一个值。
引入模板是为了:

*   *   允许搜索平台之间的通用查询管理
    *   定义复杂查询
    *   定义不能静态确定的运行时参数(如过滤器)

在下图中你可以看到三个例子:前两个是 Solr 查询的例子，而第三个是使用 Elasticsearch。

![query_templates](img/563976f8897cf58e5873008c56f080be.png)

您还可以创建多个版本文件夹(1.0 版、1.1 版、1.2 版等)，并在这些文件夹中保存不同的查询模板版本。这将为每个查询模板版本运行评估，并允许您轻松地比较查询模板的更改。

## 搜索引擎配置/外部搜索引擎

一旦定义了基本事实，就有可能为评估定义一个目标搜索系统。
RRE 的开源版本支持**嵌入一个搜索实例**(来自配置文件)和一个**外部运行实例**。

## 配置集

RRE 鼓励一种可重复的方法:跟踪你的版本，并清楚地给你的每个版本分配一个不可变的标签。通过这种方式，我们将结束我们系统的历史进程，RRE 将能够进行比较并重现评估。
配置集的实际内容实际上取决于目标搜索平台。对于 Apache Solr，每个版本文件夹(见下图)包含一个或多个 Solr 核心。相反，Elasticsearch 包含一个 JSON 文件(下图中的“index-shape.json ”),该文件包含索引设置&映射。

![configuration_sets](img/47d91522e72e694a8132bd89b0d3ee6c.png)

## 执行

运行 RRE 评估的最佳方式是使用 maven 插件，建立一个 maven 项目来导入 RRE 库并执行它们。
详细的分步指南在此链接:
[运行您的评估！](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/Maven%20Archetype)

## 输出

输出评估数据可用:

*   作为一个 **JSON 文件**:用于进一步阐述
*   作为**电子表格**:用于将评估结果交付给其他人(如商业利益相关者)
*   在一个 [**Web 控制台**](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/RRE%20Server) 中，度量及其值被实时刷新(在每次构建之后)

![](img/cb040a0da353767db738adfedbe98da3.png)

![offline search quality evaluation](img/879e4d05d1dc4a69ceb27da09397193f.png)

###### 链接

*   项目[储存库](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator)
*   项目[维基](https://web.archive.org/web/20220929231153/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki)

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做[搜索质量评估](https://web.archive.org/web/20220929231153/https://sease.io/training/search-quality-evaluation-trainings/search-quality-evaluation-training)和[搜索相关性](https://web.archive.org/web/20220929231153/https://sease.io/training/search-relevance-training/search-relevance-training-solr)培训吗？
我们还提供这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929231153/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于离线搜索质量评估的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！