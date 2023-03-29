# RRE-企业:评估浏览/比较控制板

> 原文：<https://web.archive.org/web/sease.io/2022/08/rre-enterprise-evaluation-explore-compare-dashboard.html>

这篇博客教程是为 RRE 企业的高级用户设计的。

我们已经介绍了概览仪表板，这是搜索质量评估结果的第一种方法。

如果你是一个从事搜索项目的软件工程师，你想更深入地探索搜索质量评估结果。

浏览功能使您能够调查特定指标返回特定分数的原因:

![](img/ad609e8c6588f5c9ea5ea8f932a018e7.png)

First of all, you select the evaluation of interest: the list of all the evaluations is available, each entry shows the label assigned, the unique ID, and the date.

一旦选择了评估，属于该评估的所有集合都是可选择的。
对于您在评估时选择的每个集合，每个指标显示得分以及与之前评估的差异:

![](img/4b3edfe0b5f1f6e275f86faf5e3981fa.png)

在探索结果时，您可以关注一个或多个指标，所以让我们看看如果您选择其中一些指标会发生什么:

![](img/e9b0d3d9b28622bd5ca1d79d3e63a0c7.png)

第一个视图显示了您的评级集中定义的每个主题的指标平均分。这让你了解每组查询是如何执行的。
扩展和折叠功能允许您更深入地浏览结果，查看每个单独查询的性能:

![](img/5968f3da70e3455626be159b7f1cd348.png)

单击单个查询，您会看到所有详细信息，包括搜索结果列表以及与之前评估的比较:

![](img/7258ed93c75a0d381652330e4a5973c4.png)

这使您能够深入调试为什么某些查询可能会改善/变坏，检查黑盒 API 查询、相应的搜索引擎查询(可能由 RRE 企业自动发现)和结果列表。

**注意**Explore 仪表盘会自动将最新评估与之前的评估进行比较(如果可用)。

## 比较

**比较**功能与**探索**相同，主要区别在于:

您可以选择任意两个评估进行比较。
**注意:**它们必须在用于运行评估的**等级集**方面兼容。

![](img/feffdeccdbc2827e3a9d26ce2e6c4b2f.png)

*   **目标迭代**是您想要与过去的评估进行比较的**当前评估**
*   **基础迭代**是用作控制的**过去评估**

比较两个评估的用户界面保持不变。

您现在已经完成了使用 RRE 企业版的基本教程。

下一个博客系列将探索 RRE 企业的内部结构，以及它是如何创造奇迹的！

敬请期待！

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做了一个[搜索质量评估](https://web.archive.org/web/20221202231254/https://sease.io/training/search-quality-evaluation-trainings)培训(分两个版本:针对产品经理和软件工程师)？我们也提供关于这些主题的咨询，如果您想为您的搜索项目建立搜索质量评估渠道，请联系我们！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Drop constant features:一个现实世界的学习排名场景的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！