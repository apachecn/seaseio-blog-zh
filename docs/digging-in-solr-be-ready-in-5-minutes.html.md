# 深入 Solr 代码:5 分钟指南

> 原文：<https://web.archive.org/web/sease.io/2019/12/digging-in-solr-be-ready-in-5-minutes.html>

比方说，您需要编写一个组件、一个请求处理程序，或者一些需要插入 Solr 的定制代码。或者，你需要对 Lucene/Solr 的内部有更深的理解，了解代码中实际发生的事情。

我知道:单元测试、集成测试，一切都是为了确保事情如你所愿；但是这里我说的是不同的东西:在开发时，它(至少对我来说)是一个非常有用的**生产和调试环境**,在这里可以使用短的开发迭代，一步一步地跟踪代码中发生的事情，深入了解事情在幕后是如何工作的。

根据我的经验，我发现这在一些情况下很有用:

*   **我必须写一些 Solr 插件**:在这种情况下，我想有一个开发环境，允许尽可能快地编写和调试代码
*   **我必须研究一些 Solr 内部机制**:比方说，我需要检查当一个字段同时为 docValues="true "和 stored="true "时，在检索时会发生什么；Solr 从哪里获取字段值？

【T6

让我们看看如何在几分钟内完成这两项任务！

###### 步骤 1:克隆我们的模板库

克隆以下存储库[【1】](https://web.archive.org/web/20220930005426/https://github.com/SeaseLtd/solr-addon-project-skeleton)

一旦导入到您最喜欢的 IDE 中，项目布局将如下所示:

![This image has an empty alt attribute; its file name is template-project-imported-1.png](img/80b8ab72da658ffb6c27e9080ccdf909.png)

正如您所看到的，模板项目提供了:

*   **一个定制的*令牌过滤器*** ，它在文本分析期间简单地打印出标准输出令牌。**注意这只是一个例子**(如果你想调试一个分析器，这很有用):我可以创建一个 *SearchComponent、*一个 *Tokenizer* 或者任何我需要的东西。
*   一个**样本 Solr 配置**，配置了最少的东西
*   一个**测试超类型层**(*base integration Test*)和**一个样本测试** ( *测试*)，它加载一些数据，执行一个查询，然后打印出结果。

**想不到，就这些！没有第二步了！**

###### 用例 1:实现、调试和测试一个附加组件

如前所述，在示例存储库中，我们已经有了一个简单的附加组件，它由一个*令牌过滤器*组成，它在标准输出中打印分析链中产生的每个令牌。过滤器已在 Solr 配置中声明为“文本”字段类型分析器的一部分:

```
<fieldType name="text" class="solr.TextField">
    <analyzer>
        ...
        <filter class="io.sease.labs.solr.SystemOutTokenFilterFactory"/>
    </analyzer>
</fieldType>
```

test 类触发这个分析器，因为它索引一些文档，所以如果您将它作为一个普通的 *JUnit* 测试运行，您将看到下面的输出:

```
startOffset=0,endOffset=6,positionIncrement=1,positionLength=1,type=word => Object
startOffset=7,endOffset=15,positionIncrement=1,positionLength=1,type=word => Oriented
startOffset=16,endOffset=24,positionIncrement=1,positionLength=1,type=word => Software
startOffset=25,endOffset=37,positionIncrement=1,positionLength=1,type=word => Construction
startOffset=0,endOffset=6,positionIncrement=1,positionLength=1,type=word => Design
startOffset=7,endOffset=16,positionIncrement=1,positionLength=1,type=word => Patterns:
startOffset=17,endOffset=25,positionIncrement=1,positionLength=1,type=word => Elements
startOffset=26,endOffset=28,positionIncrement=1,positionLength=1,type=word => of
startOffset=29,endOffset=37,positionIncrement=1,positionLength=1,type=word => Reusable
startOffset=38,endOffset=53,positionIncrement=1,positionLength=1,type=word => Object-Oriented
startOffset=54,endOffset=62,positionIncrement=1,positionLength=1,type=word => Software

*DOC 1* 
id = 1
title = Object Oriented Software Construction

*DOC 2* 
id = 2
title = Design Patterns: Elements of Reusable Object-Oriented Software
```

如果您在令牌过滤器中发现一个断点，并在调试模式下重新运行*测试*类，调试器将如预期的那样在该行停止:

![](img/0e6ea24a32600d1aa39946d1797fb051.png)

###### 用例 2:调试 Solr 内部

在这种情况下，没有定制代码，因为记住，目标是调查一些 Solr 内部。具体来说，这个例子中我要回答的问题是:假设我们有一个字段

```
<field name="myfield" type="string" docValues="true" stored="true"/>
```

还有一个请求

```
q=...&...&fl=myfield
```

Solr 从哪里得到[1]的字段值？

我要做的第一件事是改变项目中的一些东西:

*   *   **schema.xml** :添加上面的字段定义
    *   ***测试*类**:更改查询参数(添加 **fl=myfield** )并为索引文档中的 *myfield* 字段添加一些值。

现在，有一个前提:由于这篇博文的目标不是实际回答上面的问题，我们将跳过理解整个查询执行流和检测放置断点的正确位置所需的所有调查阶段。

经过一番调查，我们了解到*RetrieveFieldOptimizer*[2]类在该过程中起着重要作用，所以让我们打开它并放置一些断点:

![This image has an empty alt attribute; its file name is retrieveFieldOptimizer-1.png](img/48027ad14dd78f46dac494aa4d5e849c.png)

如您所见，该类的名称和意图非常清楚，但我仍然想看看运行时会发生什么:让我们在调试模式下启动*测试*类，正如所料

![This image has an empty alt attribute; its file name is debug_1.png](img/eb7d04e680258295beaf315a7969f1b0.png)

我可以看到字段“myfield”已被收集到“storedFields”集合中，而 dvFields (DocValues 字段)集合为空，即使该字段启用了 DocValues 标志。所以这可能暗示了我一些事情…



继续前进，我们到达了*优化*方法，在这里我们遇到了 SOLR-8344[【3】](https://web.archive.org/web/20220930005426/https://issues.apache.org/jira/browse/SOLR-8344)中描述的优化:

![This image has an empty alt attribute; its file name is debug_2.png](img/8cc1240ad95c0cf118f55ce62d54d22c.png)

同样，这只是一个例子，这里的目标不是描述研究结果；然而，简单地说，如果所有要求的字段

*   *   启用*文件值*和*存储的*标志
    *   不是多值的

然后 Solr 只从 docValues 中检索值**。**

新年快乐！

注意这个类(以及优化)已经在 Solr 7 中引入。[2]

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930005426/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220930005426/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930005426/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于挖掘 Solr 代码的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！