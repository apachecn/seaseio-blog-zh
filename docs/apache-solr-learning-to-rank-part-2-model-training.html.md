# Solr 正在学习如何更好地排名——第 2 部分——模型训练

> 原文：<https://web.archive.org/web/sease.io/2016/08/apache-solr-learning-to-rank-part-2-model-training.html>

## Apache Solr 学习排名的模型培训

如果你想为 Apache Solr 学习排名训练一个模型，你来对地方了。
这篇博客文章是关于 Apache Solr Learning To Rank integration 的模型培训阶段。

We modelled our dataset, collected the data and refined it in [Part 1](https://web.archive.org/web/20220930000224/https://sease.io/2016/07/apache-solr-learning-to-rank-part-1-data-collection.html).We have now a shiny lot-of-rows training set ready to be used to train the mathematical model that will re-rank the resulting documents coming out from our queries.The very first activity to carry on is to decide the model that fits best our requirements.I will focus in this blog post on two types of models, the ones currently supported by the Apache Solr Learning To Rank integration [[1]](https://web.archive.org/web/20220930000224/https://issues.apache.org/jira/browse/SOLR-8542).

## SVM 排名

Ranking SVM is a linear model based on Support Vector Machines.

SVM 排序算法是一种成对排序方法，它根据结果与特定查询的“相关性”来自适应地对结果进行排序(我们在第 1 部分看到了成对文档-查询的相关性评级的例子)。

排名 SVM 函数将每个搜索查询映射到每个样本的特征。
这个映射函数将每个数据对投影到一个特征空间。

训练数据的每个样本(查询文档)将被用于排序 SVM 算法以改进映射。

一般来说，SVM 排名在训练时包括三个步骤:

*   *   它将查询和点击结果之间的相似性映射到某个特征空间。
    *   它计算步骤 1 中获得的任意两个向量之间的距离。
    *   它形成了一个类似于标准 SVM 分类的优化问题，并使用常规 SVM 求解器来解决该问题

Given the list of the features that describe our problem, an SVM model will assign a numerical weight to each of them, the resulting function will assign a score to a document given in input the feature vector that describes it.Let’s see an example SVM Model in the Json format expected by the Solr plugin :*e.g.*

```
{
    "class":"org.apache.solr.ltr.ranking.RankSVMModel",
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
```

给定 3 个示例特征:
**userTextTitleMatch**–(查询相关特征)–(二进制)
**originalScore**–(查询相关特征)–(序数)
**is book**–(文档相关特征)–(二进制)

示例中的排名 SVM 模型构建了变量的线性函数，并为每个变量分配了不同的权重。

给定文档(用其特征向量表示):
***D1***【1.0，100，1】
***D2***【0.0，80，1】

应用该模型，我们得到分数:

**得分(D1)**= 1.0 * userTextTitleMatch(1.0)+0.5 * originalScore(100)+0.1 * is book(1.0)=**51.1**
**得分(D2)**= 1.0 * userTextTitleMatch(0.0)+0.5 * originalScore(80)+0.1 * is book(1.0)=**40.1**

**要点**:

*   *   易于调试和理解
    *   线性函数

这里可以找到一些不错的训练 SVM 模型的开源库[【2】](https://web.archive.org/web/20220930000224/https://www.csie.ntu.edu.tw/%7Ecjlin/libsvm/)[【2.1】](https://web.archive.org/web/20220930000224/https://www.csie.ntu.edu.tw/%7Ecjlin/liblinear/)。

## 兰达马特

LambdaMART 是一个基于树集合的模型。
集合的每棵树都是加权回归树，并且最终预测得分是每个回归树的预测的加权和。
回归树是一种决策树，它将特征向量作为输入，并在输出中返回标量数值。
在高层次上，LambdaMART 是一种使用梯度推进来直接优化学习以对特定成本函数(如 NDCG)进行排序的算法。

为了理解 LambdaMART，让我们来探讨主要的两个方面:Lambda 和 MART。

**MART**
LambdaMART 是梯度推进回归树的一个具体实例，也称为多重加法回归树(MART)。
梯度增强是一种用于形成模型的技术，该模型是“弱学习者”集合的加权组合。在我们的例子中，每个“弱学习者”都是一棵决策树。
**λ**
在每个训练点，我们需要提供一个梯度，使我们能够最小化成本函数(无论我们选择什么，例如 NDCG)。
为了解决这个问题，LambdaMART 使用了一个来自 lambdaRank 的想法:
在每个特定点，我们计算一个值，作为我们需要的梯度，这个组件将有效地修改排序，向上或向下推动文档组，并有效地影响叶值的规则，这些规则覆盖了用于计算 lambda 的点。
关于其他详细信息，这些资源非常有用[【3】](https://web.archive.org/web/20220930000224/https://wellecks.wordpress.com/2015/02/21/peering-into-the-black-box-visualizing-lambdamart/)[【4】](https://web.archive.org/web/20220930000224/https://www.microsoft.com/en-us/research/publication/ranking-boosting-and-model-adaptation/?from=http%3A%2F%2Fresearch.microsoft.com%2Fpubs%2F69536%2Ftr-2008-109.pdf)。

LambdaMART 目前被认为是最有效的模型之一，它已经被许多获奖的竞赛所证明。

让我们看看 lambdaMART 模型是什么样子的

```
{
    "class":"org.apache.solr.ltr.ranking.LambdaMARTModel",
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

这个例子模型由两棵树组成，每个分支分割评估一个特征值上的条件表达式，如果特征值是:
*< =阈值*我们访问左边的分支
*>阈值*我们访问右边的分支

到达一棵树的叶子将产生一个分数，该分数将根据树的权重进行加权，然后与产生的其他分数相加(该模型是树的集合)。
鉴于文件:

**D1**【1，9】
**D2**【0，10】

应用该模型，我们得到分数:

**Score(D1)**= 1 *(userTextTitleMatch(1)>0.5 go right，originalScore(9)<10 = 50)+
2 *-10 =**30**

**得分(D2)**= 1 *(userTextTitleMatch(0)<= 0.5 =-100)+
2 *-10 =**-120**

根据这一模式，D1 文件更为相关。

**要点**:

*   *   树木的组合是有效的
    *   难以调试
    *   非线性函数

有很好的开源库来训练 LambdaMART 模型，我们将在这个研究案例中使用的库是RankLib[【5】](https://web.archive.org/web/20220930000224/https://sourceforge.net/p/lemur/wiki/RankLib/)

## 评估指标

Important phase of the journey is to validate the model :How good is our model in re-ranking our dataset ?How good is the model in re-ranking unknown data  ?It is really important to carefully select the evaluation metric that best suites your algorithm and your needs.Evaluating the model will be vital for the validation phase during the training, for testing the model against new data and to consistently being able to assess the predicting quality of the model trained.**N.B**. this is particularly important: having a good understanding of the evaluation metric for the algorithm selected, can help a lot in tuning,  improving the model and finding anomalies in the training data.

#### NDCG@K

选择 lambdaMART 时，归一化贴现累积增益@k 是一个自然的拟合。
该指标评估排名模型的性能，从 0.0 到 1.0 不等，1.0
代表理想的排名模型。
以 K 为参数:
K:返回的最大文档数。

K 为每个查询指定重新排序后要评估的前 K 个结果。
**注意:**将 K 设置为您想要优化的顶部文档的数量。
通常是您在结果首页显示的文档数。

在输入测试集的情况下，通过 queryId 对其进行分组，并且对于每个查询，将理想排名(通过后代相关性对文档进行排序而获得)与由排名模型生成的排名进行比较。
计算所有查询 id 的平均值。
详细解释可以在维基百科上找到[【6】](https://web.archive.org/web/20220930000224/https://en.wikipedia.org/wiki/Discounted_cumulative_gain#Normalized_DCG)。

知道了它是如何工作的，在推理 NDCG 时有两个因素听起来非常重要:
1)每个查询 Id 在测试集中的相关性分数的分布
2)每个查询 Id 的样本数

***战争故事 1:高度相关样本太多- >高 NDCG***
给定测试集 A:
5 qid:1 1:1…
5 qid:1 1:1…
4 qid:1 1:1…
5 qid:2 1:1…
4 qid:2 1:1…
4 qid:2 1:1…

即使排名模型做得不好，相关样本的高分布也增加了在度量中达到高分的概率。

***War Story 2: small RankLists -> high NDCG***
Having few samples (<K) per queryId means that we will not evaluate the performance of our ranking model with accuracy.In the edge case of 1 sample per queryId, our NDCG@10 will be perfect, but actually this reflect simply a really poor training/test set.**N.B.** be careful on how you generate your queryId as you can end up in this edge case and have a wrong perception of the quality of your ranking model.

## 模特培训

Focus of this section will be how to train a LambdaMART model using RankLib [[5]](https://web.archive.org/web/20220930000224/https://sourceforge.net/p/lemur/wiki/RankLib/),Let’s see an example and analyse all the parameters:***java  -jar RankLib-2.7.jar ****–**train** /var/trainingSets/trainingSet_2016-07-20.txt ****-feature** /var/ltr/trainingSets/trainingSet_2016-07-20_features.txt ****-ranker** 6 ****-leaf** 75****–******mls** 5000****-metric2t** NDCG@20 **–**kcv** 5****-tvs** 0.8 ****-kcvmd** /var/models ****-kcvmn** model-name.xml****-sparse ***

| 参数 | 描述 |
| **列车**列车 | 定型集文件的路径 |
| **功能** | 特性描述文件，列出学员需要考虑的特性 id，每一个 id 占一行。如果未指定，将使用所有功能。 |
| **排名者** | 指定要使用的分级算法，例如 6: LambdaMART |
| **metric2t** | 用于优化训练数据的度量。支持:地图，NDCG@k，DCG@k，P@k，RR@k，ERR@k(默认=ERR@10) |
| **电视** | 将训练验证分割设置为(x)(1.0-x)。
x *(训练集的大小)将用于训练
(1.0 -x) *(训练集的大小)将用于验证 |
| **kcv** | 代表 k 交叉验证指定是否希望仅使用指定的训练数据执行 k 重交叉验证(默认值=NoCV)。这意味着我们将训练数据分成 k 个子集，并执行 k 次训练。
在每次执行中，1 个子集将用作测试集，k-1 个子集将用于训练。
**注意:**在 tvs 和 kcv 都被定义的情况下，首先我们为交叉验证进行拆分，然后生成的训练集将在训练/验证中进行拆分。*例如* *初始训练集大小:100 行**-kcv 5**-tvs0.8 * *-Test Set : 20 rows* *-Training Set :  64 ( 0.8 * 80)* *-Validation Set : 16 (0.2 * 80)* |
| **kcvmd** | 保存通过交叉验证训练的模型的目录(默认为不保存)。 |
| **kcvmn** | 在每个文件夹中学习模型的名称。它将以折叠号为前缀(默认为空)。 |
| **稀疏** | 允许对训练集进行稀疏数据处理(这意味着您不需要为每个样本指定所有没有值的特征) |
| **树** | 集合的树的数量(默认值=1000)。
学习问题越复杂，我们需要的树就越多，但是一定要小心，不要过拟合训练数据 |
| **叶子** | 每棵树的叶子数(默认值=100)。
根据树的数量，仔细调整该值以避免过拟合树是很重要的。 |
| **收缩** | 这是算法的学习率(默认值=0.1)
如果这过于激进，排名模型将快速适应训练集，但不会对验证集评估做出正确反应，这意味着整体模型较差。 |
| **tc** | 树分裂的候选阈值数。
-1 使用所有特征值(默认值=256)
增加该值会增加模型的复杂性和可能的过拟合。建议从一个较低的值开始，然后根据您的要素的一般基数增加它。 |
| **大联盟** | 最小叶片支持–每个叶片必须包含的最小样本数(默认值=1)。如果我们想要处理异常值，这是一个非常重要的参数。
我们可以调整该参数，使其仅包含具有大量样本的叶片，丢弃由弱支持验证的模式。 |

调整模型训练参数不是一件容易的事情。
除了一般建议之外，仔细比较评估指标得分的试验&误差是一条可行之路。
当我们用不同的调整配置生成了许多模型时，在一个通用测试集上比较不同的模型会有所帮助。

## 将模型转换为 Json

We finally produced a model which is performing quite well on our Test set.Cool, it’s the time to push it to Solr and see the magic in action.But before we can do that, we need to convert the model generated by the Machine Learning libraries into the Json format Solr expects ( you remember the section about the models supported ? )Taking as example a LambdaMART model, Ranklib will produce an xml model.So you will need to parse it and convert it to the Json format.Would be interesting to implement directly in RankLib the possibility of selecting in output the Json format expected by Solr.Any contribution is welcome, [RankLib Extension for Json Models](https://web.archive.org/web/20220930000224/https://sease.io/2022/07/rre-enterprise-evaluation-overview-dashboard.html)!In the next part, we’ll see how to visualise and understand better the model generated.This activity can be vital to debug the model, see the most popular features, find out some anomalies in the training set and actually assess the validity of the model itself.// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930000224/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和[学习排名](https://web.archive.org/web/20220930000224/https://sease.io/training/learning-to-rank-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930000224/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Solr 正在学习更好地排名的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！