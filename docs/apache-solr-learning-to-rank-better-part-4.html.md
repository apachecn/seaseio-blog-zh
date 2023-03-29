# Solr 正在学习如何更好地排名——第 4 部分——Solr 整合

> 原文：<https://web.archive.org/web/sease.io/2016/10/apache-solr-learning-to-rank-better-part-4.html>

## 旅程的最后阶段

这篇博文是关于 Apache Solr 学习排名(LTR)集成的。

我们建立了数据集模型，收集了数据，并在第一部分中对其进行了提炼。
在[第二部](https://web.archive.org/web/20220929233943/https://sease.io/2016/08/apache-solr-learning-to-rank-part-2-model-training.html)中训练模特。
分析和评估[第 3 部分](https://web.archive.org/web/20220929233943/https://sease.io/2016/08/apache-solr-learning-to-rank-part-3-ltr-tools.html)中的模型和训练集。
我们已经准备好将模型和特性定义应用到 Solr 中。
在这篇博文中，我将重点介绍来自彭博的 Apache Solr Learning To Rank ( LTR)集成[【1】](https://web.archive.org/web/20220929233943/https://issues.apache.org/jira/browse/SOLR-8542)。
该贡献已完成，可从 Apache Solr 6.4 获得。
本博客主要基于彭博贡献的自述文件[【2】](https://web.archive.org/web/20220929233943/https://github.com/bloomberg/lucene-solr/blob/master-ltr-plugin-develop/solr/contrib/ltr/README.md)。

## Apache Solr 学习排名(LTR)集成

Apache Solr Learning To Rank ( LTR)集成允许 Solr 对搜索结果进行重新排序，评估提供的 Learning To Rank 模型。
插件的主要职责是:

*   特征定义的存储
*   模型的存储
*   特征提取和缓存
*   搜索结果重新排名

## 特征定义

正如我们从以前的帖子中了解到的，特征向量是每个文档/查询对的数学表示，模型将根据该向量对每个搜索结果进行评分。
当然我们需要告诉 Solr 如何为搜索结果中的每个文档生成特征向量。
这里是特征定义文件。
一个 Json 数组，描述了通过机器学习的 LTR 模型对我们的文档进行评分所需的所有相关特征。

*例如*

```
[{ "name": "isBook",
  "class": "org.apache.solr.ltr.feature.SolrFeature",
  "params":{ "fq": ["{!terms f=category}book"] }
},
{
  "name":  "documentRecency",
  "class": "org.apache.solr.ltr.feature.SolrFeature",
  "params": {
      "q": "{!func}recip( ms(NOW,publish_date), 3.16e-11, 1, 1)"
  }
},
{
  "name" : "userTextTitleMatch",
  "class" : "org.apache.solr.ltr.feature.SolrFeature",
  "params" : { "q" : "{!field f=title}${user_text}" }
},
{
  "name":"book_price",
  "class":"org.apache.solr.ltr.feature.FieldValueFeature",
  "params":{"field":"book_price"}
},
{
  "name":"originalScore",
  "class":"org.apache.solr.ltr.feature.OriginalScoreFeature",
  "params":{}
},
{
   "name" : "userFromMobile",
   "class" : "org.apache.solr.ltr.feature.ValueFeature",
   "params" : { "value" : "${userFromMobile:}", "required":true }
}] 
```

| **土壤特征** |
| –依赖查询
–独立查询 |
| Solr 特性由 Solr sintax 之后的 Solr 查询定义。
Solr 特性的值被计算为针对我们正在评分的文档运行的查询的返回值。
此功能可以依赖于查询时间参数，也可以独立于查询(参见示例) |
| **例如**
" params ":{ " FQ ":[" {！terms f = category } book "]}
–独立查询
–布尔特征
如果文档与“类别”字段中的术语“图书”匹配，特征值将为 1。
它与查询无关，因为没有查询参数影响此计算。
**"params":{"q": "{！func}recip( ms(NOW，publish_date)，3.16e-11，1，1)" }**
–查询依赖
–序数特征
特征值将作为函数查询的结果来计算，文档越新，值越接近 1。
它依赖于查询，因为“现在”会影响特征值。
**"params":{"q": "{！field f = title } $ { user _ text } " }**
–查询相关
–序数特征
特征值将作为查询结果进行计算，用户查询的标题内容越相关，该值越高。
它依赖于查询，因为“user_text”查询参数会影响计算。 |

| **字段值特征** |
| –独立于查询 |
| 字段值特征由 Solr 字段定义。
特性的值被计算为我们正在评分的文档的字段内容。
该字段必须被存储或被文档赋值。这个特性是独立于查询的(参见示例) |
| 例如。
**" params ":{ " field ":" book_price " }**
–独立查询
–序数特征
该特征的值将是给定文档的“book _ price”字段的内容。
它与查询无关，因为没有查询参数影响此计算。 |

| **价值特征** |
| –查询级别
–常量 |
| 值特征由常量或外部查询参数定义。
特性的值计算为 solr 请求中作为 efi(外部特性信息)参数或常量传递的值。
该功能仅**依赖于**配置的参数(见示例) |
| **例如**
**【params】:{ " value ":" $ { user _ from _ mobile:****} "， " required ":false }**
–查询级别
–布尔特征
用户将传递' userFromMobile '请求参数作为 efi
该特征的值将是参数的值
如果请求中缺少参数，将分配默认值
如果需要，将引发异常如果请求中缺少参数**" params ":{ " value ":" 5****"， “必选”:false }**
–常量
–序数特征
特征值将被计算为常量值“5”。 除了常数，没有什么影响计算。 |

| **原始核心特征** |
| –依赖于查询 |
| 原始得分特征被定义为没有附加参数。
特性的值被计算为给定输入查询的文档的原始 lucene 分数。
该功能依赖于查询时间参数(参见示例) |
| **例如**
**【params】:{ }**
—查询依赖
—序数特征
特征值将是给定输入查询的原始 lucene 得分。
它依赖于查询，因为整个输入查询都会影响该计算。 |

#### 外部特征信息

正如您在特征定义 json 中注意到的，外部请求参数会影响特征提取计算。
运行重新排序查询时，可以传递将在特征提取时使用的附加请求参数。我们会在相关章节中详细了解这一点。

例如 rq={！ltr reRankDocs = 3 model = external model}

#### 部署功能定义

很好，我们定义了模型所需的所有特性，现在可以将它们发送到 Solr:

```
curl -XPUT 'http://localhost:8983/solr/collection1/schema/feature-store' --data-binary @/path/features.json -H 'Content-type:application/json' 
```

#### 查看功能定义

很好，我们定义了模型所需的所有特性，现在可以将它们发送到 Solr:

```
curl -XGET 'http://localhost:8983/solr/collection1/schema/feature-store' 
```

## 模型定义

We extensively explored how to train models and how models look like in the format the Solr plugin is expecting.For details I suggest you reading : [Part 2](https://web.archive.org/web/20220929233943/https://sease.io/2016/08/24/solr-is-learning-to-rank-better-part-2-model-training)Let’s have a quick summary anyway  :

#### 线性模型(排名 SVM，恶作剧)

*如*

```
 {
    "class":"org.apache.solr.ltr.model.LinearModel",
    "name":"myModelName",
    "features":[
        { "name": "userTextTitleMatch"},
        { "name": "originalScore"},
        { "name": "isBook"}
    ],
    "params":{
        "weights": {
            "userTextTitleMatch": 1.0,
            "originalScore": 0.5,
            "isBook": 0.1
        }
    }
} 
```

#### 多重加法树(LambdaMART，梯度增强回归树)

*例如*

```
{
    "class":"org.apache.solr.ltr.model.MultipleAdditiveTreesModel",
    "name":"lambdamartmodel",
    "features":[
        { "name": "userTextTitleMatch"},
        { "name": "originalScore"}
    ],
    "params":{
        "trees": [
            {
                "weight" : 1,
                "root": {
                    "feature": "userTextTitleMatch",
                    "threshold": 0.5,
                    "left" : {
                        "value" : -100
                    },
                    "right": {
                        "feature" : "originalScore",
                        "threshold": 10.0,
                        "left" : {
                            "value" : 50
                        },
                        "right" : {
                            "value" : 75
                        }
                    }
                }
            },
            {
                "weight" : 2,
                "root": {
                    "value" : -10
                }
            }
        ]
    }
} 
```

#### 启发式增强模型(实验)

启发式助推模型是一种实验模型，将线性助推结合到任何模型中。
目前在实验分支[【3】](https://web.archive.org/web/20220929233943/https://github.com/bloomberg/lucene-solr/tree/master-ltr-plugin-experimental/solr/contrib/ltr)。
目前只有*org . Apache . Solr . ltr . ranking . heuristicboostedlambdamartmodel*支持此功能。
这种方法背后的原因是，有时，在训练时，我们没有在查询时使用的所有功能。
例如
您的训练集不是建立在搜索结果的点击量上，并且包含遗留数据，但是您希望将原始得分作为提升因子包含在内
我们来看看详细配置:
给定:

```
"features":[ { "name": "userTextTitleMatch"}, { "name": "originalScoreFeature"} ]
"boost":{ "feature":"originalScoreFeature", "weight":0.1, "type":"SUM" } 
```

原始得分特征值(加权系数为 0.1)将被添加到 LambdaMART 树产生的得分中。

```
 "boost":{ "feature":"originalScoreFeature", "weight":0.1, "type":"PRODUCT" } 
```

加权系数为 0.1 的原始得分特征值将乘以 LambdaMART 树产生的得分。

**注意**使用这种方法时要格外小心。这为分数计算引入了手动提升，当您没有太多数据用于训练时，这增加了灵活性。然而，您将失去机器学习模型的一些好处，该模型被优化以重新排列您的结果。当你得到更多的数据并且你的模型变得更好时，你应该停止手动增强。

例如

```
{
    "class":"org.apache.solr.ltr.ranking.HeuristicBoostedLambdaMARTModel",
    "name":"lambdamartmodel",
    "features":[
        { "name": "userTextTitleMatch"},
        { "name": "originalScoreFeature"}
    ],
    "params":{
    "boost": {
          "feature": "originalScoreFeature",
          "weight": 0.5,
          "type": "SUM"
        },
        "trees": [
            {
                "weight" : 1,
                "root": {
                    "feature": "userTextTitleMatch",
                    "threshold": 0.5,
                    "left" : {
                        "value" : -100
                    },
                    "right": {
                        "value" : 10}
 }
 },
 {
 "weight" : 2,
 "root": {
 "value" : -10
 }
 }
 ]
 }
 }
```

#### 部署模型

正如我们在特性定义中看到的，部署模型非常简单:

```
curl -XPUT 'http://localhost:8983/solr/collection1/schema/model-store' --data-binary @/path/model.json -H 'Content-type:application/json' 
```

#### 视图模型

该模型将存储在一个易于访问的 json 存储中:

```
curl -XGET 'http://localhost:8983/solr/collection1/schema/model-store'
```

## 重新排序查询

要使用机器学习的 LTR 模型对搜索结果进行重新排序，需要使用 Apache Solr Learning To Rank ( LTR)查询解析器调用 rerank 组件。

查询重新排序允许您运行初始查询(A)来匹配文档，然后重新排序前 N 个文档，并基于第二个查询(B)对它们重新评分。
由于来自查询 B 的代价更高的排序仅应用于前 N 个文档，因此与仅使用复杂查询 B 本身相比，它对性能的影响更小——代价是使用简单查询 A 得分很低的文档在重新排序阶段可能不会被考虑，即使它们使用查询 B 得分很高。 *Solr Wiki*

Apache Solr Learning To Rank ( LTR)集成定义了一个额外的查询解析器，可用于定义重新排序策略。
特别是，当在搜索结果中重新记录文档时:

*   *   从文档中提取特征
    *   相对于提取的特征向量评估模型来计算分数
    *   最终的搜索结果将根据新的分数重新排序

rq={!ltr model=myModelName reRankDocs=25}

**！ltr**–将使用 ltr 查询解析器
**model = my model name**–指定使用模型存储中的哪个模型对文档进行评分
**reRankDocs = 25**–指定仅对原始排名中的前 25 个搜索结果进行评分和重新排序

当传递将用于提取特征向量的外部特征信息(EFI)时，语法非常相似:

rq={！ltr reRankDocs = 3 model = external model EFI . parameter 1 = ' value 1 ' EFI . parameter 2 = ' value 2 ' }

例如

rq={！ltr rerandocs = 3 model = external model EFI . user _ input _ query = ' casa Blanca ' EFI . user _ from _ mobile = 1 }

#### 分片

当使用分片时，每个分片都将被重新排序，因此每个分片都将考虑重新排序的文档。

**例如**
10 个碎片
你用:
rq={！ltr reRankDocs=10 …
您将获得总共 100 个重新排序的文档。

#### 页码

分页是微妙的。

让我们探索一下单个 Solr 节点和分片架构上的场景。

##### 单一 Solr 节点

reRankDocs=15rows=10This means each page is composed of 10 results.
What happens when we hit page 2?
The first 5 documents in the search results will have been rescored and affected by the reranking.
The latter 5 documents will preserve the original score and original ranking.e.g.Doc 11 – score= 1.2Doc 12 – score= 1.1Doc 13 – score= 1.0Doc 14 – score= 0.9Doc 15 – score= 0.8Doc 16 – score= 5.7Doc 17 – score= 5.6
Doc 18 – score= 5.5
Doc 19 – score= 4.6
Doc 20 – score= 2.4This means that score(15) could be < score(16), but document 15 and 16 are still in the expected order.
The reason is that the top 15 documents are rescored and reranked and the rest is left unchanged.

##### 分片建筑

reRankDocs=15rows=10
Shards number=2

当查找第 2 页时，Solr 将触发对 she 碎片的查询，以收集每个碎片的 2 页:
碎片 1 : 10 个重新排序的文档(第 1 页)+ 10 个原始分类的文档(第 2 页)
碎片 2 : 10 个重新排序的文档(第 1 页)+ 10 个原始分类的文档(第 2 页)

结果将被合并，并且可能，原始评分的搜索结果可以在重新排序的文档之上。

一个可能的解决方案是将分数标准化，以防止重新排序的结果被原始分数超过的可能性。

**注意**:问题会在你到达 rows * page > reRankDocs 后发生。在 reRankDocs 非常高的情况下，这个问题只会在深度分页中出现。

## 特征提取和缓存

Extracting the features from the search results document is the most onerous task while reranking using LTR.The LTRScoringQuery will take care of computing the feature values in the feature vector and then delegate the final score generation to the LTRScoringModel.
For each document, the definitions in the feature store are applied to generate the vector.The vector can be generated in parallel, leveraging a multi-threaded approach.Extra care must be taken into account when configuring the number of threads in the game.The features vector is currently cached in toto in the QUERY_DOC_FV cache.This means that given the query and EFIs, we cache the entire feature vector for the document.

简单地在输入中给出一个不同的 EFI 请求参数将意味着一个不同的特征向量 hashcode，从而使缓存的 hashcode 无效。

通过分别管理独立于查询、依赖于查询和查询级别特性的缓存，可以潜在地改进这一点[【5】](https://web.archive.org/web/20220929233943/https://issues.apache.org/jira/browse/SOLR-10448)。

[1] [Solr LTR 插件](https://web.archive.org/web/20220929233943/https://issues.apache.org/jira/browse/SOLR-8542)
【2】[Solr LTR 插件官方自述](https://web.archive.org/web/20220929233943/https://github.com/bloomberg/lucene-solr/blob/master-ltr-plugin-develop/solr/contrib/ltr/README.md)
【3】[Solr LTR 插件实验分支](https://web.archive.org/web/20220929233943/https://github.com/bloomberg/lucene-solr/tree/master-ltr-plugin-experimental/solr/contrib/ltr)
【5】[缓存改进](https://web.archive.org/web/20220929233943/https://issues.apache.org/jira/browse/SOLR-10448)

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929233943/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和[学习排名](https://web.archive.org/web/20220929233943/https://sease.io/training/learning-to-rank-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929233943/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Solr 正在学习更好地排名的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！