# 评级排名评价者:帮助穷人(搜索工程师)

> 原文：<https://web.archive.org/web/sease.io/2018/07/rated-ranking-evaluator.html>

软件工程师总是被要求向他的客户提供关于可交付产品质量的具体证据。搜索工程师处理这种通用软件质量的专门化，它被称为搜索质量。

什么是搜索质量？为什么它在搜索基础设施中如此重要？毕竟，“软件质量”应该是无所不包的，它应该总是包括一切(实际上是这样)，但是当我们处理搜索系统时，质量是一个非常抽象的术语，很难预先定义。

搜索基础设施的功能正确性(假设正确性是影响系统质量的唯一因素——它不是——自然与人的判断和观点相关联，不幸的是，我们知道人们的观点可能不同。

将从搜索系统中获得价值的业务涉众可以属于不同的类别，可以有不同的期望，并且他们可以对期望的系统正确性有不同的想法。

在这种情况下，搜索工程师在选择方面面临许多挑战，最后，他必须提供关于这些选择的功能覆盖的具体证据。

这就是我们开发分级评定员(以下简称 RRE)的背景。

## 这是什么？

评级评估器(RRE)[【1】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator)是一个搜索质量评估工具，用于评估来自搜索基础设施的结果的质量。

这有助于搜索工程师的日常工作。你是搜索工程师吗？您是否正在调整/实施/更改/配置搜索基础架构？你想有什么东西给你一个证据来证明变化之间的改进吗？RRE 可以帮你一把。

RRE 将搜索系统满足用户信息需求的程度正式化，在“技术”层面，将丰富的树状域模型与几个评估指标相结合，但也在“功能”层面，提供可以针对商业利益相关者的人类可读输出。

它鼓励在搜索系统的开发和演变过程中采用增量/迭代/不可变的方法:假设我们从 x.y 版本开始我们的系统:当需要对其配置应用一些相关的更改时，而不是对 x.y 应用更改，最好克隆它并将那些更改应用到新的全新版本。

通过这种方式，RRE 将对所有可用版本执行评估过程，它将提供后续版本之间的差异/趋势，因此您可以立即获得系统在相关性方面的精细画面。

这篇文章只是对 RRE 的一个简要总结。你可以在项目 Wiki[【2】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki)中找到更详细的信息。

## 简而言之，我能从 RRE 得到什么？

您可以将 RRE 配置为项目构建周期的复合部分。这意味着，每次触发构建时，都会执行一个评估过程。

RRE 并不局限于一个给定的搜索平台:它提供了一个迷你框架[【3】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/Search%20Platform%20Framework)用于插入不同的搜索平台。目前我们有两个可用的绑定:Apache Solr 和 elastic search[【4】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/Supported%20Versions)。

输出评估数据将可用:

*   *   作为 **JSON 文件**:进一步阐述
    *   作为**电子表格**:用于将评估结果交付给其他人(如商业利益相关者)
    *   在 **Web 控制台**[【5】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/RRE%20Server)中，指标及其值会实时刷新(在每次构建之后)

![](img/677e7810c320b0e9382d7d7f6fe36e7c.png)

## 它是如何工作的

RRE 提供了一个丰富的、复合的、树状的领域模型[【6】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/Domain%20Model)，其中的评估概念可以在不同的层次上看到。

![RRE Domain Model](img/de9d25f76605a9d18ce06afcad5d2071.png)

顶层的**评估**只是嵌套实体的容器。注意，所有的实体关系都是一对多的**。在这种情况下，**语料库**被定义为测试数据集。RRE 将使用它来执行评估过程；在一个评估过程中，您可以拥有多个数据集。**

**主题是信息需求:它从最终用户的角度定义了功能需求。在一个主题中，我们可以有几个查询，它们表达了相同的需求，但更接近于技术层。RRE 在中间提供了进一步的抽象:查询组。**查询组**是一组应该产生相同结果的查询(因此与相同的判断集相关联)。**

**查询是 RRE 领域模型的技术叶子，它被进一步分解成几个角度，每个角度对应我们系统的一个可用版本。当然，查询本身是一个单独的实体，但是在评估会话期间，它的具体执行会发生几次，每次针对一个可用版本。这是因为 RRE 需要衡量所有版本的搜索结果(即查询执行)。**

**对于每个版本，我们最终都会有一个或多个指标，这取决于配置。最后但同样重要的是，即使度量是在查询/版本级别计算的，RRE 也将在更高的级别聚合这些值(参见图中的垂直虚线)，因此域模型中的每个实体/级别将提供所有可用度量的聚合视角(即，我可能对给定查询的 NDCG 感兴趣，或者我可以只在主题级别停止我的分析)。**

 **## 投入

为了执行评估流程，RRE 需要以下东西:

*   *   一个或多个**语料库** / **测试集**[【7】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/What%20We%20Need%20To%20Provide#corpus):这些是特定领域的代表性数据集，将用于填充和查询目标搜索平台
    *   一个或多个**配置集**[【8】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/What%20We%20Need%20To%20Provide#configuration-sets):虽然没有什么反对拥有一个单一的配置，但是至少需要两个版本，以便提供评估措施之间的比较。
    *   一个或多个**评级集**[【9】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/What%20We%20Need%20To%20Provide#ratings):这是根据每个查询组的相关文档定义判断的地方。

## 输出

RRE 的具体输出取决于它运行的运行时容器。RRE 核心本身只是一个库，所以当在项目中以编程方式使用时，它输出一组对应于上述域模型的对象。

当它作为 Maven 插件使用时，主要以 JSON 格式[【10】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator/wiki/The%20Evaluation%20Output)输出相同的结构。这些数据然后被用于产生进一步的输出，比如电子表格。同样的有效载荷可以发送到另一个名为 RRE 服务器的模块，它提供了一个基于 AngularJS 的 web 控制台，可以自动刷新。

当我们围绕某个问题进行内部迭代/尝试时，RRE 控制台非常有用，这通常需要非常短的*编辑并立即检查*周期。想象一下，如果你的桌子上有几个显示器:第一个是你最喜欢的 IDE，你可以在那里修改东西，运行构建。第二个是 RRE 控制台(见下文)。每次构建完成后，只需在控制台上看一下，就可以立即得到您所做更改的反馈。

![](img/ecafc38b85fbc2f12d94ddd5749a9866.png)

![](img/9f9757fcc472f57069528c8218788034.png)

## 我能从哪里开始？

Github 中的项目资源库[【1】](https://web.archive.org/web/20220930005630/https://github.com/SeaseLtd/rated-ranking-evaluator)提供了您需要的一切:关于它如何工作以及如何快速开始使用 RRE 的详细文档。

如果您需要帮助，请随时联系我们！我们感谢任何反馈，建议，最后但同样重要的是，贡献。

## 未来作品

可想而知，这个话题是相当庞大的。我们有很多关于平台进化的有趣想法。

以下是一些例子:

*   *   与一些工具集成，用于建立相关性判断。这可能是一些用户界面或一个更复杂的用户交互收集器[【11】](https://web.archive.org/web/20220930005630/https://www.slideshare.net/AndreaGazzarini/search-quality-evaluation-a-developer-perspective/30)(它将在计算的在线指标如点击率和销售率的基础上自动生成评级集)
    *   詹金斯插件[【12】](https://web.archive.org/web/20220930005630/https://www.slideshare.net/AndreaGazzarini/search-quality-evaluation-a-developer-perspective/31):为了更好地将 RRE 集成到流行的 CI 工具中
    *   Gradle 插件
    *   Apache Solr 等级评估 API[【13】](https://web.archive.org/web/20220930005630/https://www.slideshare.net/AndreaGazzarini/search-quality-evaluation-a-developer-perspective/32):使用 RRE 核心，我们可以在 Solr 中实现一个等级评估端点，类似于 Elasticsearch 中提供的等级评估 API[【14】](https://web.archive.org/web/20220930005630/https://www.elastic.co/guide/en/elasticsearch/reference/6.3/search-rank-eval.html)
    *   ？？？其他？热烈欢迎任何建议

###### 相关链接

*   我们在伦敦 Apache Solr/Lucene 会议上演讲的幻灯片
*   亚麻对伦敦会议的精彩总结[【16】](https://web.archive.org/web/20220930005630/http://www.flax.co.uk/blog/2018/06/29/lucene-solr-london-search-quality-testing-and-search-procurement)

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做[搜索质量评估](https://web.archive.org/web/20220930005630/https://sease.io/training/search-quality-evaluation-trainings/search-quality-evaluation-training)和[搜索相关性](https://web.archive.org/web/20220930005630/https://sease.io/training/search-relevance-training/search-relevance-training-solr)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930005630/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于评级评估员的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**