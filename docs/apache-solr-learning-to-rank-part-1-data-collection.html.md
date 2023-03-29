# Solr 正在学习如何更好地排名——第 1 部分——数据收集

> 原文：<https://web.archive.org/web/sease.io/2016/07/apache-solr-learning-to-rank-part-1-data-collection.html>

## 介绍

这篇博客文章是关于在 Apache Solr 搜索引擎中学习排名的必要旅程。

Learning to Rank [[1]](https://web.archive.org/web/20220929234108/https://en.wikipedia.org/wiki/Learning_to_rank) is the application of Machine Learning in the construction of ranking models for Information Retrieval systems.Introducing supervised learning from user behaviour and signals can improve the relevancy of the documents retrieved bringing a new approach in ranking them.Can be helpful in countless domains, refining free text search results or building a ranking algorithm where only filtering is happening and no initial scoring is available.

这一系列博客文章探索了从 Signals 集合到 Apache Solr 中的重新排序的完整旅程。

**第 1 部分**将探索数据收集、数据建模和提炼阶段
[**第 2 部分**](https://web.archive.org/web/20220929234108/https://sease.io/2016/08/apache-solr-learning-to-rank-part-2-model-training.html) 将探索培训阶段
[**第 3 部分**](https://web.archive.org/web/20220929234108/https://sease.io/2016/08/apache-solr-learning-to-rank-part-3-ltr-tools.html) 将描述一套实用程序来分析您的模型
**[第 4 部分](https://web.archive.org/web/20220929234108/https://sease.io/2016/10/apache-solr-learning-to-rank-better-part-4.html)** 将涵盖 Solr 集成

**要求**:对机器学习的最少知识和兴趣

特别感谢迭戈·切卡雷利，他在 2015 年给了我很大的帮助，让我了解了这个令人惊叹的话题。
再次特别感谢 **Juan Luis Andrades** ，感谢我们在这次旅程中分享的激情，以及 **Jesse McLaughlin** 对 java 技术的细致洞察。最后特别感谢大卫·本伯里，他以兴趣、激情和深思熟虑的想法为这项事业做出了贡献。

## 收集信号

The start of the journey is the signals collection, it is a key phase and involves the modelling of the supervised training set that will be used to train the model.A training set can be **Explicit** or **Implicit**.

###### 含蓄的

An Implicit training set is collected from user behaviours and interactions.***e.g**.**Historical sales and transactions of an E-commerce website**User Clicks in the search result page**Time spent on each document accessed*A training set of this type is quite noisy but allows to collect great numbers of signals with small effort.More the user was engaged with a particular document, stronger the signal of relevancy.***e.g**.**A sale of a product is a stronger signal of relevancy than adding it to the basket**User Clicks in the search result page in comparison with documents shown but not clicked**Longer you read a document, stronger the relevancy**+ **Pros** : Cheap to build**– **Cons** : Noisy*

###### *显式*

An Explicit training set is collected directly from the interaction with Human Experts.Given a query and a set of documents, the Human expert will rate the relevancy of each document in the result set.The score assigned to the document will rate how relevant the document was for the query.To remove the subjective bias is suggested to use a team of experts to build the training set.A training set of this type is highly accurate but it is really expensive to build as you need a huge team of experts to produce thousands of rating for all the queries of interest.*+ Pros : Accuracy**– Cons : Expensive to build*

###### 训练集

The training set can have different forms, for the sake of this post we will focus on an algorithm that internally uses a  pairwise/listwise approach.In particular the syntax exposed will be the RankLib [[2]](https://web.archive.org/web/20220929234108/https://sourceforge.net/p/lemur/wiki/RankLib%20File%20Format/) syntax.Each sample of the training set is a signal/event that describes a triple (Query-Document-Rating).Let’s take as an example a query (1) and a document (1A)

3 qid:1 1:1 2:1 3:0 # 1A

The document can be represented as a feature vector which is a vector of scalar numeric values.Each element in the vector represents a specific aspect of the document.We’ll see this in the feature part in details.Let’s now focus in understanding the sample :

3
文档与查询的相关程度

qid:1
识别查询

1:1 2:1 3:0
文档，表示为数字特征的向量

# 1A
发表评论，让你的训练数据更具可读性

## 特征工程

For the convenience of Machine Learning algorithms, query-document pairs are represented by numerical vectors.Components of such vectors are called features and can be divided into different groups ( if they depend on the document, the query or both) :**Document Features (query independent)**This kind of feature depends only on the document and not on the query.*e.g.**Document length**Price of the product**User Rating of the product*An interesting aspect of these features is that they can be potentially precomputed in off-line mode during indexing. They may be used to compute the document’s *static quality score* (or *static rank*), which is often used to speed up search query evaluation.**Query Dependent Features **This kind of features depends on the query and on the documente.g.*Is the document containing the query text in the title?*
*Is the document (product) of the same brand as expressed in the query***Query Level Features **This kind of features depends only on the query.e.g.*Number of words in the query**Cuisine Type Selected( e.g. “Italian”, “Sushi” when searching for Restaurants)*
*Date Selected ( e.g. when searching in a Hotel Booking system)**Department Selected ( e.g. “electonics”, “kitchen”, “DIY” … in an E-commerce website)***User Dependent Features**Also in this case this kind of feature does not depend on the document.It only depends on the user running the query.*e.g.**Device **Age of the user**Gender*As described, for convenience of the mathematical algorithms each high-level feature must be modelled as a numeric feature.In the real world, a feature describes an aspect of the object (document) and must be represented accordingly:**Ordinal Features**An ordinal feature represents a numerical value with a certain position in a sequence of numbers.*e.g.**Star Rating  ( for a document describing a Hotel)**Price  ( for a document describing an e-commerce product)*For the Star Rating feature, stands an order for the different values:1<2<3<4<5  is logically correct.For the Price feature, the same observation applies.100$ < 200$ <300$A feature is Ordinal when it is possible to compare different values and decide the ranking of these.**Categorical Features**A categorical feature represents an attribute of an object that have a set of distinct possible values.In computer science, it is common to call the possible values of a categorical features Enumerations.*e.g.**Colour ( for a document describing a dress)**Country ( for a document describing a location)*It easy to observe that to give an order to the values of a categorical feature does not make any sense.For the Colour feature :red < blue < black has no general meaning.**Binary Features**A binary feature represents an attribute of an object that can have only two possible values.Traditionally 0 / 1 in accordance with the binary numeral system.*e.g.**Is the product available? yes/no ( for a document describing an e-commerce product)**Is the colour Red? ( for a document describing a dress)**Is the country Italy ? ( for a document describing a location)*

#### 一个热编码

当分类特征描述您的文档时，很容易将每个类别表示为一个整数 Id:

*e.g.****Categorical Feature**: colour****Distinct Values**: red, green, blue****Representation** : colour:1, colour:2 colour:3 *With this strategy, the Machine learning algorithm will be able to manage the feature values…But is the information we pass, the same as the original one?Representing a categorical feature as an ordinal feature is introducing an additional ordinal relationship :1(red) < 2(green) < 3(blue)which doesn’t reflect the original information.There are different ways to encode categorical features to make them understandable by the training algorithm. We need basically to encode the original information the feature provides in a numeric form, without any loss or addition if possible.One possible approach is called One Hot Encoding [[3]](https://web.archive.org/web/20220929234108/https://www.quora.com/What-is-one-hot-encoding-and-when-is-it-used-in-data-science):Given a categorical feature with N distinct values, encode it in N binary features, each feature will state if the category applies to the Document.*e.g.****Categorical Feature**: colour****Distinct Values**: red, green, blue****Encoded Features** : colour_red, colour_green, colour_blue *A document representing a blue shirt will be described by the following feature vector :… colour_red:0 colour_green:0 colour_blue:1 …One Hot Encoding is really useful to properly model your information, but take care of the cardinality of your categorical feature as this will be reflected in the number of final features that will describe your signal.***War Story 1: High Cardinality Categorical Feature***A signal describing a document with a high-level categorical feature (with N distinct values) can produce a Feature vector of length N.This can deteriorate the performance of your trainer as it will need to manage many more features per signal.It actually happened to me, that simply adding one categorical feature was bringing in thousands of binary features, exhausting the hardware my trainer was using,  killing the training process.To mitigate this, can be useful to limit the encoded distinct values only to a subset :

*   *   白名单/黑名单方法由业务驱动
    *   仅保留最高出现值
    *   仅保留出现次数超过阈值的值
    *   将其余部分编码为特殊特征:colour_misc
    *   将不同的值散列到一组精简的散列中

#### 特征标准化

Feature Normalisation is a method used to standardize the range of values across different features, a technique quite useful in the data pre-processing phase.As the majority of machine learning algorithms use the Euclidean distance to calculate the distance between two different points (training vector signals), if a feature has a widely different scales, the distance can be governed by this particular feature.Normalizing can simplify the problem and give the same weight to each of the features involved.There are different type of normalization, some of them :

*   *   线性归一化(基于最小值/最大值)
    *   总和归一化(基于要素的所有值的总和)
    *   z 得分(基于特征的平均值/标准差)

#### 特征值量化

Another approach to simplify the job of the training algorithm is to quantise the feature values, in order to reduce the cardinality of distinct values per feature.It is basically the simple concept of rounding, whenever we realise that it does not make any difference for the domain to model the value with high precision, it is suggested to simplify it and round it to the acceptable level.*e.g.****Domain**: hospitality****Ranking problem**: Rank restaurants documents**Assuming a feature is the trip_advisor_reviews_count, is it really necessary to model the value as the precise amount of reviews? Normally would be simpler to round to the nearest k ( 250 or whatever sensible to the business)*
*Note: Extra care must be taken into account if following this approach.*
*The reason is that adding an artificial rounding to the data can be dangerous, we can basically compromise the feature itself setting up hard thresholds.*
*It is always better if the algorithm decides the thresholds with freedom.*
*It is suggested were possible to not quantise, or quantise only after a deep statistical analysis on our data.*
*If the target is to simplify the model for any reason, it is possible to evaluate at training time less threshold candidates, depending on the training algorithm.*

#### 缺少值

Some of the signals we are collecting could miss some of the features ( data corruption, bug in the signal collection or simply the information was not available at the time ) .Modelling our signals with a sparse feature vector will imply that a missing feature will actually be modelled as a feature with value 0.This should generally be ok, but we must be careful in the case that 0 is a valid value for the feature.e.g.*Given a **user_rating** feature**A rating of 0 means the product has a very bad rating.**A missing rating means we don’t have a rating for the product ( the product can still be really good) *A first approach could be to model the 0 ratings as slightly greater than 0 (i.e. 0 + ε) and keep the sparse representation.In this way, we are differentiating the information but we are still modelling the wrong ordinal relationship : Missing User Rating  (0) < User Rating 0 (0 + ε)Unfortunately, at least for the RankLib implementation, a missing feature will always be modelled with a value 0, this, of course, will vary from algorithm to algorithm.But we can enforce the learning a bit, adding an additional binary feature that states that the User Rating is actually missing :user_rating:0 , user_rating_missing:1 .This should help the learning process to actually understand better the difference.
Furthermore, if possible we can help the algorithm, avoiding the sparse representation if necessary and setting for the missing feature a value which is the avg of the feature itself across the different samples.

#### 极端值

Some of the signals we are collecting could have some outliers ( some signal with an unlikely extremely different value for a specific feature).This can be caused by bugs in the signal collection process or simply the anomaly can be a real instance of a really rare signal.Outliers can complicate the job of the model training and can end up in overfitting models that have difficulties in adaptation for unknown datasets.Identify and resolve anomalies can be vitally important if your dataset is quite fragile.Tool for data visualisation can help in visualising outliers, but for a deep analysis I suggest to have a read of this interesting blog post[ [4]](https://web.archive.org/web/20220929234108/https://machinelearningmastery.com/how-to-identify-outliers-in-your-data/).

#### 特征定义

Defining the proper set of features to describe the document of our domain is an hard task.It is not easy to identify in the first place all the relevant features even if we are domain experts, this procedure will take time and a lot of trial and error.Let’s see a guideline to try to build a feature vector as best as possible :

*   *   **保持简单**:从一组有限的特征开始，这些特征是描述你的问题的真正基础，产生的模型会很差，但至少你有基线。
    *   **迭代训练模型:**每次执行时删除或添加一个特征。这很费时间，但可以让你清楚地识别哪些功能真正重要。
    *   **可视化重要特征**:训练完模型后，使用可视化工具验证哪个特征出现得更多
    *   **与业务部门会面**:与业务部门会面，比较他们期望看到的重新排序和模型实际重新排序的内容。当有不一致时，让人类来解释为什么，这应该驱动去识别缺失的特征或以错误的意义使用的特征。

## 数据准备

我们仔细设计了领域文档的矢量表示，确定了信号的来源，建立了训练集。
到目前为止，一切顺利……
但是模特的表现仍然很差。
在这种情况下，原因有很多:

*   *   信号质量差(噪音)
    *   不完全特征向量表示
    *   每个查询的相关-不相关文档分布不均匀

让我们探索一些准则来克服这些困难:

#### 噪声消除

In the scenario of implicit signals, it is likely we model the relevancy rating based on an evaluation metric of the user engagement with the document given a certain query.Depending of the domain we can measure the user engagement in different ways.Let’s see an example for a specific domain :  **E-commerce**We can assign the relevancy rating of each signal depending on the user interaction :Given a scale 1 to 3 :**1** – User clicked the product**2** – User added the product to the basket**3** – User bought the productThe simplest approach would be to store 1 signal per user interaction.User behavioural signals are noisy by nature, but this approach introduces even more noise, as for the same feature vector we introduce discordant signals, specifically we are telling the training algorithm that given that feature vector and that query, the document is at the same time :vaguely relevant – relevant – strongly relevant .This doesn’t help the training algorithm at all, so we need to find a strategy to avoid that.One possible way is to keep only the strongest signal per document-query per user episode .In the case of a user buying a product, we avoid storing in the training set 3 signals, but we keep only the most relevant one.In this way we transmit to the training algorithm the only the important information for the user interaction with no confusion.

#### 不平衡数据集

In some domain, would be quite common to have a very unbalanced dataset.A dataset is unbalanced when the relevancy classes are not represented equally in the dataset i.e. we have many more samples of a relevancy class than another.Taking again the E-commerce example, the number of relevant signals (sales) will be much less than the number of weak signals (clicks).This unbalance can make the life harder to the training algorithm, as each relevant signal can be covered by many more weakly relevant ones.Let’s see how we can manipulate the dataset to partially mitigate this problem :

###### 收集更多数据

This sounds simple, but collecting more data is generally likely to help.Of course there are domain when collecting more data is not actually beneficial ( for example when the market change quite dinamically and the previous years dataset becomes almost irrelevant for predicting the current behaviours ) .

###### 对数据集进行重新采样

You can manipulate the data you collected to have more balanced data.This change is called sampling your dataset and there are two main methods that you can use to even-up the classes: Oversampling and Undersampling [[5]](https://web.archive.org/web/20220929234108/https://en.wikipedia.org/wiki/Oversampling_and_undersampling_in_data_analysis).You can add copies of instances from the under-represented relevancy class, this is called over-sampling, oryou can delete instances from the over-represented class, this technique is called under-sampling.These approaches are often very easy to implement and fast to run. They are an excellent starting point.Some ideas and suggestion :

*   *   当您有大量数据(数万或数十万个实例或更多)时，可以考虑欠采样测试，您可以随机欠采样或遵循任何可用的业务逻辑
    *   考虑测试不同的重采样比率(没有必要以 1:1 的比率为目标)
    *   当过采样考虑先进的方法，而不是简单的复制已经存在的样本时，人工产生新的样本可能是好的

请注意，重采样并不总是有助于您的训练算法，因此请使用您的用例进行详细实验。

***War Story 2: Oversampling by duplication****Given a dataset highly unbalanced, the model trained was struggling to predict accurately the desired relevancy class for document-query test samples.**Oversampling was a tempting approach, so here we go!**As I am using cross validation, the first approach has been to oversample the dataset by duplication.**I took each relevancy class and duplicate the samples until I built a balanced dataset.**Then started the training in Cross Validation and I trained a model which was immense and almost perfectly able to predict the relevancy of validation and test samples.**Cool ! I got it !**Actually was not an amazing result at all, because of course applying cross validation on an oversampled dataset built validation and test sets oversampled as well.**This means that it was really likely that a sample in the training set was appearing exactly the same in the validation set and in the test set.**The resulting model was basically highly overfitted and not that good to predict unknown test sets.**So I moved to a manual training set- validation set – test set split and oversampled only the training set.**This was definitely better and built a model that was much more suitable.**It was not able to perfectly predict validation and test sets but this was a good point as the model was able to predict unknown data sets better.**Then I trained again, this time the original dataset, manually split as before but not oversampled.**The resulting model was actually better than the oversampled one.**One of the possible reasons is that the training algorithm and model I was using (LambdaMART) didn’t get any specific help from the resampling, actually the model lost the capability of discovering which samples were converting better ( strong relevant signals : weak relevant signals ratio).**Practically I favoured the volume over the conversion ratio, increasing the recall but losing the precision of the ranker.****Conclusion** : Experiment, evaluate the approach with your algorithm, compare, don’t assume it is going to be better without checking*

#### 查询 Id 哈希

As we have seen in the initial part of the blog, each sample is a document-query pair, represented in a vectorial format.The query is represented by an Id, this Id is used to group samples for the same query, and evaluate the ranker performance over each samples group.This can give us an evaluation of how good the ranker is performing on average on all the queries of interest.This brings to carefully decide how we generate the query identifier.If we generate a **too specific hash**, we risk to build small groups of samples, this small groups can get an high score when ranking them, biased by their small size.*Extreme case**e.g.**Really specific hash, brings many groups to be 1 sample groups.**This brings up the evaluation metric score, as we are averaging and a lot of groups, being of size 1, are perfectly easy to rank.*If we generate an hash that is not specific enough we can end up in immense groups, not that helpful to evaluate our ranking model on the different real world scenarios.The ideal scenario is to have one query Id per query category of interest, with a good number of samples related, this would be the perfect dataset, in this way we can validate both :

*   *   组内相关性(因为组由足够的样本组成)
    *   查询的平均值(因为我们有一组有效的不同查询可用)

The query category could depend on a set of Query Dependent Features, this means that we can calculate the hash using the values of these features.Being careful we maintain a balance between the group sizes and the granularity of the hash.It is important to have the query categories across the training/validation/test set :*e.g.**We have 3 different query categories , based on the value of a user dependent feature ( user_age_segment) .**These age segments represents three very different market segments, that require very different ranking models.**When building our training set we want enough samples for each category and we want them to be split across the training/validation/test sets to be able to validate how good we are in predicting the different market segment.**This can potentially drive to build separate models and separate data sets if it is the case.*// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929234108/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和[学习排名](https://web.archive.org/web/20220929234108/https://sease.io/training/learning-to-rank-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929234108/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于学习 Apache Solr 排名的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！