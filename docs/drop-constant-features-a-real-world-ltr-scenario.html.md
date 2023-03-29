# 删除常数特性:现实世界的学习排名场景

> 原文：<https://web.archive.org/web/sease.io/2021/06/drop-constant-features-a-real-world-ltr-scenario.html>

这篇“提示和技巧”只是想建议您在训练学习排序(LTR)模型之前，在管道末端仔细检查您的数据集特征。

数据预处理的一个基本部分是**特征选择，这被认为是任何机器学习管道中最耗时的部分之一。**去除多余特征，只保留必要的相关特征用于模型构建的过程。

在所有观察值/行中具有相同值的数据集中的列被称为**常量特征**，并被视为冗余数据。他们的移除会影响你的模型性能吗？不要！那些包含常量值的特征在预测您的响应时无法提供任何有用的信息，因此最好将它们从数据集中移除，这不仅是为了加快训练速度，还因为如果管理不善，它们可能会导致数值问题和错误(例如在编码阶段)。

这似乎是显而易见的，但是在所有的数据预处理步骤(即数据收集、数据建模和细化阶段)之后，您可能会再次得到一些不变的特性(甚至没有意识到！)那是应该忽略的。

这就是为什么我想向您展示一个在现实世界实施过程中可能发生的场景。

## 案例研究示例

让我们想象一个学习排序的场景。

我们必须管理一个电子商务产品目录。我们收集用户与网站产品的所有互动(例如，浏览、点击、加入购物车、销售..)并创建一个由<query document="">对组成的数据集(例如用户选择的过滤器(*查询级*)和查看/点击/销售的产品的所有特征(文档级))。为了简单起见，这里只给出了一个示例，展示了一些功能:</query>

![](img/73bfa063a98a9f2b6a154547b64d45ed.png)

为了训练我们的 LTR 模型，我们仔细设计了领域文档的矢量表示，并精确构建了我们的训练集，以使模型尽可能好地执行。

大多数特征必须被操纵；特别是为了使分类特征能够被机器学习算法理解，我们基本上需要将该特征提供的原始信息编码成数字形式，如果可能的话，没有任何损失。对分类特征进行编码有不同的方式，一种可能的方法称为一次热编码[【1】](https://web.archive.org/web/20220929235710/https://machinelearningmastery.com/why-one-hot-encode-data-in-machine-learning/):
给定一个具有 N 个不同值的分类特征，将其编码为 N 个二进制特征，每个特征将声明该类别是否适用于该文档。



假设您有如下所示的多值分类特征:

![](img/eec441df8de0e700c47166a06e4e6694.png)

这个特性名为' **productCategories** '，由一个整数数组组成；对于每个交互，它包含产品所属的所有类别。使用一次性编码，每个 N 个不同的值将通过向数据集中添加 N 个列进行编码，如下表所示。数据集中的所有分类要素都会出现这种情况。此外，我们使用所有查询级别的特性来生成 query_ID ( [查看关于它的这篇博客文章](https://web.archive.org/web/20220929235710/https://sease.io/2021/03/a-learning-to-rank-project-on-a-daily-song-ranking-problem-part-3.html))。

![](img/1bb13f7d7c7f04fe2a941970319ec735.png)

在将数据集分成训练集和测试集之前，我们使用了一种称为*clean _ data _ frame _ from _ single _ label*的方法，该方法删除了所有只有一个相关性标签的 query_ID(阅读 [Part3](https://web.archive.org/web/20220929235710/https://sease.io/2021/03/a-learning-to-rank-project-on-a-daily-song-ranking-problem-part-3.html) 和 [Part4](https://web.archive.org/web/20220929235710/https://sease.io/2021/05/a-learning-to-rank-project-on-a-daily-song-ranking-problem-part-4.html) 的博客文章可以了解原因)。

可能发生的情况是，query_ID = 3(上表的第三行)可能在该阶段被潜在地移除，因为属于该查询组的所有观察具有相同的相关性标签(例如等于 1)。我们以下面的数据框结束:

![](img/20936751af6f44e76400156e50e07278.png)

您可以很容易地看到，我们已经向数据集添加了两个不必要的要素:这些要素的所有行都是假的，因为包含 True 的一行(或多行)已被 clean _ data _ frame _ from _ single _ label 方法删除。

因此，功能 productCategories_3483 和 productCategories_4639 无意中变成了常量列，而且正如已经说过的，我们应该忽略在定型模型之前无法从中获得任何信息的列。



###### 删除常数列方法

我们需要在拆分阶段之前添加一个方法来检查和删除所有这些特征:

```
def drop_constant_columns(dataframe):
    constant_columns = [col for col in dataframe.columns if len(dataframe[col].unique()) == 1]
    dataframe_after_drop = dataframe.drop(constant_columns, axis=1)
    logging.debug('- - - - Number of constant features found : {:d}'.format(len(constant_columns)))
    logging.debug('- - - - Constant features removed :' + str(constant_columns))
    percentage = len(constant_columns) / len(dataframe.columns)
    if percentage > 0.50:
        logging.warning('**** WARNING - 50% of the features have been removed')
    return dataframe_after_drop
```

###### **模特培训**

为了证明到目前为止所解释的内容，我们已经训练了一个学习排序模型，一次是通过保留不变的特征，另一次是通过从数据集中删除它们。在实验中，我们使用了从普通电子商务中收集的交互日志数据。预处理部分之后的数据集总共包含 6035 个特征。

【T9

我们的方法发现了 825 个恒定特征(大约 14%)。我们使用 LambdaMART[【2】](https://web.archive.org/web/20220929235710/https://medium.com/rocket-travel/why-we-chose-lambdamart-for-our-hotel-ranking-model-45f84e22cec)训练了两个模型:保持和去除不变的特征。

![](img/e169b7336d577cc5700bdf1020416cff.png)

我们可以确认性能没有差异；恒定特征的存在(或不存在)对目标没有影响，事实上，度量 ndcg@10 在两个模型中是相同的。如果我们删除不变的特征，模型训练的持续时间将减少 6 分钟。在时间、内存和其他技术问题(编码)方面，我们只能从它们的移除中受益，特别是如果它们很多的话。

要删除常量功能，您可能还对其他方法感兴趣，请查看以下链接:



*   *   sk learn:sk learn . feature _ selection。变异阈值[【3】](https://web.archive.org/web/20220929235710/https://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.VarianceThreshold.html)
    *   H20: 'ignore_const_cols '参数 [[4]](https://web.archive.org/web/20220929235710/https://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/algo-params/ignore_const_cols.html)
    *   r:drop _ constant _ cols[【5】](https://web.archive.org/web/20220929235710/https://rdrr.io/cran/hutils/man/drop_constant_cols.html)

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做[学习排名](https://web.archive.org/web/20220929235710/https://sease.io/learning-to-rank-training)和 [Apache Solr 初学者](https://web.archive.org/web/20220929235710/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929235710/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Drop constant features:一个现实世界的学习排名场景的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！