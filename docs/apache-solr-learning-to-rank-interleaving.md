# Apache Solr 学习等级交错

> 原文：<https://web.archive.org/web/sease.io/2020/12/apache-solr-learning-to-rank-interleaving.html>

**学习排序**[【1】](https://web.archive.org/web/20220929233900/https://en.wikipedia.org/wiki/Learning_to_rank#cite_note-liu-1)是机器学习，通常是监督、半监督或强化学习，在构建信息检索系统的排序模型中的应用。
训练数据集由<查询、文档>对的列表组成，每个列表中的项目之间指定了一些偏序。
这种顺序通常是通过对每个项目给出数字或顺序分数或二元判断(例如“相关”或“不相关”)来得出的。
排名模型旨在进行排名，即以与训练数据中的排名类似的方式，在新的、看不见的查询和搜索结果中产生项目的排列。

维基百科(一个基于 wiki 技术的多语言的百科全书协作计划ˌ也是一部用不同语言写成的网络百科全书ˌ 其目标及宗旨是为全人类提供自由的百科全书)ˌ开放性的百科全书

## 学习在 Apache Solr 中排名

学习排名在 2017 年初用 Apache Solr 6.4.0 接近了 Apache Solr[【2】](https://web.archive.org/web/20220929233900/https://lucene.apache.org/solr/guide/8_7/learning-to-rank.html)。
彭博[【3】](https://web.archive.org/web/20220929233900/https://www.techatbloomberg.com/blog/bloomberg-integrated-learning-rank-apache-solr/)的贡献实现了一个新的重新排序查询解析器 **ltr** ，它使用机器学习模型来重新排序原始查询的前 K 个搜索结果:

```
.../query?q=test&rq={!ltr model=myModel reRankDocs=100}&fl=id,score
```

示例中的查询从 Apache Solr 中检索查询文本“test”的文档，并对前 100 名进行重新排序(最初按原始 Solr 分数排序)。
用于重新排序的关联函数是“myModel ”,它对应于用户上传的定制模型，并使用专门的机器学习库进行训练。
最终返回给用户的前 100 个搜索结果按照“myModel”排序。
为了更好地理解 Apache Solr Learning To Rank integration 的工作原理，我们发布了一系列详细描述这一点的博客文章:

*   *   [Solr 正在学习如何更好地排名——第 1 部分——数据收集](https://web.archive.org/web/20220929233900/https://sease.io/2016/07/apache-solr-learning-to-rank-part-1-data-collection.html)
    *   [Solr 正在学习如何更好地排名——第 2 部分——模型培训](https://web.archive.org/web/20220929233900/https://sease.io/2016/08/apache-solr-learning-to-rank-part-2-model-training.html)
    *   [Solr 正在学习如何更好地排名——第三部分——Ltr 工具](https://web.archive.org/web/20220929233900/https://sease.io/2016/08/apache-solr-learning-to-rank-part-3-ltr-tools.html)
    *   [Solr 正在学习如何更好地排名——第 4 部分——Solr 整合](https://web.archive.org/web/20220929233900/https://sease.io/2016/10/apache-solr-learning-to-rank-better-part-4.html)
    *   [解释学习用树形排列模型](https://web.archive.org/web/20220929233900/https://sease.io/2020/07/explaining-learning-to-rank-models-with-tree-shap.html)

另外，官方的 Apache Solr 参考指南提供了许多细节和用例[【7】](https://web.archive.org/web/20220929233900/https://lucene.apache.org/solr/guide/8_7/learning-to-rank.html)

## 在线评估在学习排名中的重要性

在线评估用于估计适合特定信息检索系统的最佳排序模型。
它通过对用户行为的解释来比较排名功能，用户行为由收集到的与我们正在评估的系统的交互直接表示。
这叫做隐性反馈。
在线测试有许多优点，我们在[在线测试在学习排名中的重要性-第 1 部分](https://web.archive.org/web/20220929233900/https://sease.io/2020/04/the-importance-of-online-testing-in-learning-to-rank-part-1.html)中探讨了这些优点。

在这篇博文中，我们重点关注交错评估，特别是团队草案交错(TDI)方法，这是 Solr 8.8(2021 年 1 月 29 日发布)中的一个新功能。

## 交叉

交错是一种信息检索系统的在线评估方法，它通过混合结果和解释用户的隐式反馈来比较排序函数。
交错是 AB 测试的替代方法。它避免了与将用户分成两组(一组暴露于控制系统，另一组暴露于变体)相关的主要差异源，以及结果的随后组合。
[学习排名在线测试:交错](https://web.archive.org/web/20220929233900/https://sease.io/2020/05/online-testing-for-learning-to-rank-interleaving.html)详细探讨了这个话题。

## 团队草稿交错

团队选秀交错考虑两种排名模式: **modelA** 和 **modelB** 。对于一个给定的查询，每个模型创建它的文档排序列表 La = (a1，a2，…)和 Lb = (b1，b2，…)。然后，该算法创建唯一的排序列表 I 以返回给用户。
该列表由 Chapelle 在[【4】](https://web.archive.org/web/20220929233900/https://www.cs.cornell.edu/people/tj/publications/chapelle_etal_12a.pdf)中描述的两个列表 La 和 Lb 的交织元素创建。列表 I 被返回给用户，用户与感兴趣的搜索结果进行交互。每个结果都是从两个排名模型中选择的，因此我们可以计算出两个模型中哪一个的点击量更高。

## Apache Solr 实现

在 Apache Lucene/Solr 提交人 Alessandro Benedetti 的工作下，Sease[【5】](https://web.archive.org/web/20220929233900/https://github.com/apache/lucene-solr/commit/af0455ac8366d6dba941f2b2674ed2a8245c76f9)对开源社区做出了贡献。
特别感谢来自彭博的 Christine Poerschke，她在文章的最后阶段提供了大量帮助，提供了详尽的评论和许多深刻的见解。

###### 可从 Apache Solr 8.8 获得

它允许运行一个重新排序的查询，传递两个模型来交错。

当前功能:

*   *   仅支持**团队草稿交错**(欢迎贡献其他算法)[【6】](https://web.archive.org/web/20220929233900/https://issues.apache.org/jira/browse/SOLR-15015)
    *   交错两种学习排序模型
    *   将一个学习排序模型与原始 Solr 排序交错
    *   获取搜索结果所来自的模型
    *   **debug=results** 返回与搜索结果所来自的模型一致的分数解释
    *   特征记录器转换器返回适当的特征值，由拾取的模型使用

## 运行交叉两个模型的重新排序查询

要对查询的结果重新排序，交叉两个模型(myModelA，myModelB)向您的搜索添加`rq`参数，在输入中传递两个模型，例如:

```
.../query?q=test&rq={!ltr model=myModelA model=myModelB reRankDocs=100}&fl=id,score
```

要获得交错为搜索结果选择的模型，在重新排序期间计算，将`[interleaving]`添加到`fl`参数，例如:

```
.../query?q=test&rq={!ltr model=myModelA model=myModelB reRankDocs=100}&fl=id,score,[interleaving]
```

Solr 响应将包括为每个搜索结果挑选的模型，类似于下面显示的输出:

```
{
  "responseHeader":{
    "status":0,
    "QTime":0,
    "params":{
      "q":"test",
      "fl":"id,score,[interleaving]",
      "rq":"{!ltr model=myModelA model=myModelB reRankDocs=100}"}},
  "response":{"numFound":2,"start":0,"maxScore":1.0005897,"docs":[
      {
        "id":"GB18030TEST",
        "score":1.0005897,
        "[interleaving]":"myModelB"},
      {
        "id":"UTF8TEST",
        "score":0.79656565,
        "[interleaving]":"myModelA"}]
  }}
```

## 运行将模型与原始排名交错的重新排名查询

当使用交错进行搜索质量评估时，将模型与原始排名进行比较可能是有用的。要对查询结果重新排序，将模型与原始排序交错，将`rq`参数添加到您的搜索中，传递特殊的内置 _ `*OriginalRanking*` _ 模型标识符作为一个输入，传递您的比较模型作为另一个模型，例如:

```
.../query?q=test&rq={!ltr model=_OriginalRanking_ model=myModel reRankDocs=100}&fl=id,score
```

要获得交错为搜索结果选择的模型，在重新排序期间计算，将`[interleaving]`添加到`fl`参数，例如:

```
.../query?q=test&rq={!ltr model=_OriginalRanking_ model=myModel reRankDocs=100}&fl=id,score,[interleaving]
```

Solr 响应将包括为每个搜索结果挑选的模型，类似于下面显示的输出:

```
{
  "responseHeader":{
    "status":0,
    "QTime":0,
    "params":{
      "q":"test",
      "fl":"id,score,[features]",
      "rq":"{!ltr model=_OriginalRanking_ model=myModel reRankDocs=100}"}},
  "response":{"numFound":2,"start":0,"maxScore":1.0005897,"docs":[
      {
        "id":"GB18030TEST",
        "score":1.0005897,
        "[interleaving]":"_OriginalRanking_"},
      {
        "id":"UTF8TEST",
        "score":0.79656565,
        "[interleaving]":"myModel"}]
  }}
```

## 如何投稿

要不要贡献一个新的交织算法？你只需要:

*   *   在一个新类中实现 Solr/contrib/ltr/src/Java/org/Apache/Solr/ltr/interleaving/interleaving . Java 接口[【7】](https://web.archive.org/web/20220929233900/https://github.com/apache/lucene-solr/blob/master/solr/contrib/ltr/src/java/org/apache/solr/ltr/interleaving/Interleaving.java)
    *   在包中添加新算法[【8】](https://web.archive.org/web/20220929233900/https://github.com/apache/lucene-solr/tree/master/solr/contrib/ltr/src/java/org/apache/solr/ltr/interleaving/algorithms)
    *   在 org . Apache . Solr . ltr . interleaving . interleaving # get implementation 中添加新的算法引用

## 限制

*   *   不支持[分布式搜索]分片

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做[学习排名](https://web.archive.org/web/20220929233900/https://sease.io/learning-to-rank-training)和 [Apache Solr 初学者](https://web.archive.org/web/20220929233900/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)培训吗？
我们还提供这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929233900/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 学习等级交错的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！