# Solr 正在学习如何更好地排名——第 3 部分——Ltr 工具

> 原文：<https://web.archive.org/web/sease.io/2016/08/apache-solr-learning-to-rank-part-3-ltr-tools.html>

## Apache Solr 学习排名-
事情变得严重了

这篇博客文章是关于 Apache Solr 学习排名工具的:一套工具来简化 Apache Solr 学习排名集成的使用。

The model has been trained in [Part 2](https://web.archive.org/web/20220929230240/https://sease.io/2016/08/apache-solr-learning-to-rank-part-2-model-training.html), we are ready to deploy it to Solr, but first it would be useful to have a better understanding of what we just created.
A LambdaMART model in a real world scenario is a massive ensemble of regression trees, not the most readable structure for a human.More we understand the model, easier will be to find anomalies and to fix/improve it.But the most important benefit of having a clearer picture of the training set and the model is the fact that it can dramatically improves the communication with the business layer :

*   *   我们领域中最重要的特性是什么？
    *   根据模型什么样的文档应该得分高？
    *   为什么这个文档(特征向量)得分那么高？

这些只是例子，但是很多类似的问题会出现，我们需要工具来回答。

## Apache Solr 学习排序工具

This is how the Learning To Rank tools project [[1]](https://web.archive.org/web/20220929230240/https://github.com/alessandrobenedetti/ltr-tools) was born ( LTR stands for Learning To Rank ).The target of the project is to use the power of Apache Solr to visualise and understand a Learning To Rank model.It is a set of simple tools specifically thought for LambdaMart models, represented in the Json format supported by the Bloomberg Apache Solr Learning To Rank integration.
Of course it is open source so feel free to extend it by introducing additional models and functionalities.All the tools provided are meant to work with a Solr backend in order to index data that we can later search easily.The tools currently available provide the support to :

*   *   在 Solr 集合中索引模型
    *   索引 Solr 集合中的训练集
        打印 LambdaMART 模型中得分最高的叶子

#### 准备

要使用学习评级(LTR)工具，您必须遵循以下简单步骤:

*   *   **设置 Solr 后端**——这将是一个新的 Solr 实例，有两个集合:**模型**，**训练集**，简单的配置可以在: *ltr-tools/configuration* 中找到
    *   **gradle build**–这会将可执行文件 jar 打包到:*ltr-tools/ltr-tools/build/libs*

#### 使用

让我们简单看一下可执行命令行界面的参数:

| 参数 | 描述 |
| **-帮助** | 打印帮助消息 |
| **-刀具** | 要执行的工具(可能值):
–模型索引器
–训练设置器
–topScoringLeavesViewer |
| **-solrURL** | 用于搜索后端的 Solr 基本 URL |
| **-型号** | model.json 文件的路径 |
| **-topK** | 要返回的最高得分叶数(按得分后代排序) |
| **-训练设置** | 定型集文件的路径 |
| **-特性** | feature-mapping.json 的路径。该文件包含要素 Id 和要素名称之间的映射。 |
| **-范畴特征** | 包含分类要素名称列表的文件的路径。 |

**注意:** *下面所有的例子都假设输入的模型是一个 LambdaMART 模型，彭博 Solr 插件期望的 json 格式。*

## 模型索引器

**要求**:后端 Solr 采集< **型号** >必须启动&运行

模型索引器是一个在 Solr 中索引 lambdaMART 模型的工具，以更好地可视化树集合的结构。
特别是，该工具会将属于 lambdaMART 系综的树的每个分支索引为 Solr 文档。
让我们来看看 solr 模式:

*配置/Solr/模型/配置*

```
 `<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false"/>  
 <field name="modelName" type="string" indexed="true" stored="true"/>  
 <field name="feature" type="string" indexed="true" stored="true" docValues="true"/>  
 <field name="threshold" type="double" indexed="true" stored="true" docValues="true"/>  
 ...` 
```

所以输入一个 lambdaMART 模型:

**例如 lambdaMARTModel1.json**

```
 `{  
   "class":"org.apache.solr.ltr.ranking.LambdaMARTModel",  
   "name":"lambdaMARTModel1",  
   "features":[  
    {  
      "name":"feature1"  
    },  
    {  
      "name":"feature2"  
    }  
   ],  
   "params":{  
    "trees":[  
      {  
       "weight":1,  
       "root":{  
         "feature":"feature1",  
         "threshold":0.5,  
         "left":{  
          "value":80  
         },  
         "right":{  
          "feature":"feature2",  
          "threshold":10.0,  
          "left":{  
            "value":50  
          },  
          "right":{  
            "value":75  
          }  
         }  
       }  
      }  
    ]  
   }  
 }` 
```

**注意:**树枝分叉是指树分成两个分支:

```
 `"feature":**"feature2"**,   
      "threshold":**10.0**,   
      "left":{   
       "value":50   
      },   
      "right":{   
       "value":75   
      }` 
```

分割发生在特征值的阈值上。
我们可以使用工具来启动索引过程:

```
`java -jar ltr-tools-1.0.jar -tool modelIndexer -model /models/lambdaMARTModel1.json  -solrURL` `http://localhost:8983/solr/models`
```

索引过程完成后，我们可以访问 Solr 并开始搜索！
*例如
这个查询将返回每个特性的响应:*

*   *   *特征出现在分支分割处的次数*
    *   *该特性的前 10 个发生阈值*
    *   *出现在该特性模型中的唯一阈值的数量*

```
 http://localhost:8983/solr/models/select?indent=on&q=*:*&wt=json&facet=true&json.facet={  
      Features: {  
           type: terms,  
           field: feature,  
           limit: -1,  
           facet: {  
                Popular_Thresholds: {  
                     type: terms,  
                     field: threshold,  
                     limit: 10  
                },  
                uniques: "unique(threshold)"  
           }  
      }  
 }&rows=0&fq=modelName:lambdaMARTModel1 
```

让我们看看如何解释 Solr 响应:

```
 `facets": {  
   "count": 3479,` //number of branch splits in the entire model "Features": {  
     "buckets": [  
       { 
         "val": "product_price",  
         "count": 317, //the feature "product_price" is occurring in the model in 317 splits "uniques": 28, //the feature "product_price" is occurring in the splits with 28 unique threshold values `"Popular_Thresholds": {  
           "buckets": [  
             {  
               "val": "250.0",` //threshold value `"count": 45` //the feature "product_price" is occurring in the splits 45 times with threshold "250.0" `},  
             {  
               "val": "350.0",  
               "count": 45  
             },  
             ...` 
```

## 训练集索引器

**要求**:后端 Solr 采集< **训练集** >必须启动&运行

训练集索引器是一个在 Solr 中对学习排序训练集(RankLib 格式)进行索引的工具，以更好地可视化数据。
特别是，该工具会将 trainign 集合的每个训练样本索引为 Solr 文档。
让我们看看 Solr 模式:

***配置/Solr/models/conf***

```
<field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false"/>
<field name="relevancy" type="tdouble" indexed="true" stored="true" docValues="true"/> 
<dynamicField name="cat_*" type="string" indexed="true" stored="true" docValues="true"/>
<dynamicField name="*" type="tdouble" indexed="true" stored="true" docValues="true"/> 
```

如你所见，这里的要点是动态字段的定义。
事实上我们事先并不知道特性的名称，但是我们可以区分分类特性(可以作为字符串索引)和顺序特性(可以作为 double 索引)。

我们现在需要 3 个输入:

**1)**rank lib 格式的训练集:

**例如:training1.txt**

```
1 qid:419267 1:300 2:4.0 3:1 6:1
4 qid:419267 1:250 2:4.5 4:1 7:1
5 qid:419267 1:450 2:5.0 5:1 6:1
2 qid:419267 1:200 2:3.5 3:1 8:1 
```

**2)** 将特征 Id 翻译成人类可读特征名称的特征映射

例如 features-mapping1.json

```
{"1":"product_price","2":"product_rating","3":"product_colour_red","4":"product_colour_green","5":"product_colour_blue","6":"product_size_S","7":"product_size_M","8":"product_size_L"} 
```

**注意事项**。映射必须是单行的 json 对象

该输入文件是可选的，可以直接将特征 id 作为名称进行索引。

**3)** 分类特征列表

*例如类别特征 1.txt*

```
product_colour
product_size 
```

该列表(每行一个要素)将向工具阐明哪些要素是分类要素，从而将类别索引为要素的字符串值。
该输入文件是可选的，可以将分类特征索引为二进制热编码特征。

要开始索引过程，请执行以下操作:

```
`java -jar ltr-tools-1.0.jar -tool trainingSetIndexer -trainingSet /trainingSets/training1.txt -features /featureMappings/feature-mapping1.json -categoricalFeatures /feature/categoricalFeatures1.txt -solrURL 
http://localhost:8983/solr/trainingSet`
```

索引过程完成后，我们可以访问 Solr 并开始搜索！
*例如
作为响应，该查询将返回所有经过过滤然后在相关性字段上进行分面的训练样本。*
*这可以是训练集的特定子集中的相关性分数分布的指示*

```
http://localhost:8983/solr/trainingSet/select?
indent=on&q=*:*&wt=json&fq=cat_product_colour:red&rows=0&facet=true&facet.field=relevancy
```

**注意:这是一种快速探索训练集的肮脏方式。我强烈建议你把它作为一个快速资源。高级数据绘图更适合可视化大数据和识别模式。**

## 最高分离开观众

最高得分叶查看器是一个工具，用于打印模型中最高得分叶的路径。
由于有了这个工具，回答类似于
“一个文档(特征向量)应该是什么样子才能得到高分？”该工具将简单地访问模型中的所有树，并跟踪每片树叶的得分。

所以输入一个 lambdaMART 模型:

*例如 lambdaMARTModel1.json*

```
 {  
   "class":"org.apache.solr.ltr.ranking.LambdaMARTModel",  
   "name":"lambdaMARTModel1",  
   "features":[  
    {  
      "name":"feature1"  
    },  
    {  
      "name":"feature2"  
    }  
   ],  
   "params":{  
    "trees":[  
      {  
       "weight":1,  
       "root":{  
         "feature":"feature1",  
         "threshold":0.5,  
         "left":{  
          "value":80  
         },  
         "right":{  
          "feature":"feature2",  
          "threshold":10.0,  
          "left":{  
            "value":50  
          },  
          "right":{  
            "value":75  
          }  
         }  
       }  
      }, ...  
    ]  
   }  
 } 
```

要启动该流程，请执行以下操作:

```
 `java -jar ltr-tools-1.0.jar -tool topScoringLeavesViewer -model /models/lambdaMARTModel1.json -topK 10` 
```

这将打印得分最高的 10 片叶子(在树中带有相关路径):

```
`1000.0 -> feature2 > 0.8, feature1 <= 100.0
200.0 -> feature2 <= 0.8, 
80.0 -> feature1 <= 0.5, 
75.0 -> feature1 > 0.5, feature2 > 10.0, 
60.0 -> feature2 > 0.8, feature1 > 100.0, 
50.0 -> feature1 > 0.5, feature2 <= 10.0,` 
```

## 结论

The Apache Solr Learning To Rank tools are quick and dirty solutions to help people understanding better and working better with Learning To Rank models.They are far from being optimal but I hope they will be helpful for people working on similar problems.Any contribution, improvement, bugfix is welcome !// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929230240/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和[学习排名](https://web.archive.org/web/20220929230240/https://sease.io/training/learning-to-rank-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929230240/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Solr 正在学习更好地排名的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**