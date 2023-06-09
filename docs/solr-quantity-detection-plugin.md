# 给身高一个合适的权重:Apache Solr 中的数量检测

> 原文：<https://web.archive.org/web/sease.io/2018/04/solr-quantity-detection-plugin.html>

## 数量检测？什么是数量？为什么我们需要检测它？

马丁·福勒(Martin Fowler)在他的《分析模式》(Analysis Patterns)[【1】](https://web.archive.org/web/20220929233733/https://martinfowler.com/books/ap.html)中所描述的一个量，被定义为一个量和单位(如 30 升，0.25 cl，或 140 cm)相结合的一对。在基于搜索的应用程序中，在许多情况下，您可能希望使用维度属性对可搜索数据集进行分类，因为这些数量在您所处理的业务环境中具有特殊的意义。我想到的第一个例子？

![Apache Solr Quantity Detection Plugin](img/fd77f865e827b3540108d697fd8773be.png)

啤酒以多种容器(如罐、瓶)提供；每一种都有多种尺寸可供选择(例如 25 厘升、50 厘升、75 厘升或 0.25 升、0.50 升、0.75 升)。一个好的目录应该在专门的字段中捕获这些信息，比如“容器”(瓶子、罐子)和“容量”(上面例子中的 25cl、50cl、75cl):这样搜索逻辑就可以正确地使用它们。分面(和随后的过滤)是一个很好的例子，说明了用户在执行了第一次搜索后可以做什么:他可以过滤和提炼结果，希望找到他想要的东西。

但是如果我们从用户交互的起点开始，根本没有结果:只有用户将要输入内容的空白文本字段。“某物”可以是任何东西，任何与他想找的产品相关的东西(在他的脑海中):品牌、容器类型、型号名称、数量。简而言之:代表他所寻找的产品的一个或多个相关特征的任何东西。

因此，在实现搜索逻辑时，主要的挑战之一是了解输入术语的含义。这通常是一个非常困难的话题，通常涉及复杂的东西(例如，机器学习)，但有时事情会向更容易的一面发展，特别是当我们想要检测的概念遵循一个共同的规则模式时:像一个量。

我们在 Sease 开发的数量检测插件[【2】](https://web.archive.org/web/20220929233733/https://github.com/SeaseLtd/solr-quantities-detection-qparsers)背后的主要思想如下:从用户输入的查询开始，首先，它检测数量(即金额和相应的单位)；然后，这些信息将从主查询中分离出来，并用于提升与这些数量相关的所有产品。这里的相关性可以有不同的含义:

*   *   **精确匹配**:所有容量为 25cl 的瓶子
    *   **范围匹配**:所有容量在 50cl 到 75cl 之间的瓶子。
    *   **等效精确匹配**:所有容量为 0.5 升(1lt = 100cl)的瓶子
    *   **等效范围匹配**:所有容量在 0.5 到 1 升之间的瓶子(1lt = 100cl)

以下是所有支持功能的简短描述:

*   *   **变体**:一个单元可以有一个首选形式和(可选)几个变体。这可以包括相同单位的不同形式(例如，mt、meter)或不同公制中的等效单位(例如，cl、once)
    *   **当量**:可以定义一个当量表，以便在运行时转换单位(“啤酒 0.25 升”与“啤酒 25 升”含义相同)。等价表用换算系数映射一个单位。
    *   **助推**:每个单位可以有一个专用的助推，特别是对多个匹配单位的加权有用。
    *   **范围**:每个单元可以有一个配置的间隙，触发范围查询，检测量可以在生成范围的中间(枢轴)、开始(最小)或结束(最大)
    *   **多字段**:如果我们有多个属性使用相同的单位(如高度、宽度、深度)
    *   **假设**:如果检测到“孤儿”金额(即没有单位的金额)，可以定义一个假设表，让 Solr 猜测单位。

请随意尝试，如果您认为它有用，请与我们分享您的想法和/或您的反馈。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929233733/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929233733/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929233733/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 中数量检测的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！