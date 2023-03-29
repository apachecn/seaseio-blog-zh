# RRE 企业报:如何建立你的目标搜索引擎

> 原文：<https://web.archive.org/web/sease.io/2022/06/rre-enterprise-how-to-set-up-your-target-search-engine.html>

我们已经在之前的[博客教程](https://web.archive.org/web/20221202232314/http://rre-enterprise-how-to-manage-your-ratings/)中看到了如何定义黄金真理，这是搜索质量评估的一个基本里程碑。
本博客重点介绍评估对象的设置:你基于 **Elasticsearch** 或 **Solr** 之上的搜索引擎。
**RRE 企业**支持两种模式:



![](img/b11507f6e8006e2be213d6e603eeb620.png)

*   *   **嵌入式** : RRE 企业使用提供的配置和测试数据，与 Docker 一起构建了一个**elastic search**/**Solr**实例。该实例将在运行评估所需的时间内可用，然后关闭。注意:你需要在你的机器上安装 docker
    *   **外部** : RRE 企业指向一个现有的**elastic search**/**Solr**实例。
        **注意:**该实例必须可以通过网络从 RRE 企业服务器访问

## 嵌入式配置上传

配置**嵌入式**方法时，您需要提供搜索引擎配置。
这些配置取决于您的搜索引擎实现( **Elasticsearch** 或 **Solr** )。
例如
–Solr config . XML，schema.xml，synonyms.txt 等用于**Solr**
–映射参数和设置用于 **Elasticsearch**

T45

**RRE 企业**支持这种配置的直接上传；所以你只需要提供所有配置的压缩文件夹；



![](img/2af3ac573c89130347fae90a4de9adb0.png)

**视频:目标搜索引擎 1**

## 来自 Git 的嵌入式配置

用 [Git](https://web.archive.org/web/20221202232314/https://git-scm.com/) 对你的搜索引擎配置进行源代码控制是一个很好的实践。通过这种方式，你可以逐步调整你的配置，跟踪不同的版本。
**RRE 企业**让您了解了这个场景:您可以直接从您的 Git 存储库、特定的分支和特定的提交中获取配置。

![](img/475afb891c02ffe8224887352ae0de3d.png)

一旦获得了存储库的唯一标识符，即 URL，并设置了一个 **RRE 企业**用来以编程方式访问存储库的公钥，您就可以开始工作了，这将在评估时用于获取您的配置的特定版本。



**视频:目标搜索引擎 2**

## 嵌入式查询模板

当使用**嵌入式**方法时，中间不会有黑盒搜索 API(假设您的前端直接与最终搜索引擎 **Elasticsearch** 或 **Solr** 对话)。
**RRE-企业**旋转搜索引擎实例，因此当评估它时，它将直接运行来自评级集的查询。
为了简化 JSON 评级文件，我们可以定义一组**查询模板**，其中包含查询结构( **Elasticsearch** DSL 查询或者 **Solr** 查询)。
模板在评级中被引用，并将被 **RRE 企业**用于构建查询，以便在评估时针对目标搜索引擎运行。
**注意:**查询模板与特定的搜索引擎配置版本相关联，您可以在上传它们时指定该版本

![](img/e3c53f31f3bb38698f8f279299cfb4bb.png)

On the left: JSON rating file, on the right: examples of query templates.

```
{
  "query": {
    "multi_match": {
      "query": "$query",
      "fields": ["title","overview","cast.name","directors.name"]
    }
  }
}
```

```
multi_match_with_titles_and_starring.json
```

**视频:目标搜索引擎 3**

## 外部的

很多时候你的前端并不直接和 Elasticsearch 或者 Solr 对话:**中间有一个搜索 API 层**。
这个 search-API 构建任何种类的 **Elasticsearch** 或 **Solr** 查询，它可能包含非常复杂的业务逻辑。
我们称之为**黑盒搜索 API** :因为**RRE-企业**不需要知道这样的中间层的实现细节是什么，也不需要知道所涉及的编程语言，它只需要知道用来运行评估的 REST 端点。
**注意:****RRE-企业**仅支持 **REST 黑盒搜索 API**

T56

当将您的评估目标配置为外部搜索引擎时，您需要确保您配置了与此类搜索引擎实例对话所需的所有信息 **RRE 企业**。
用这种方法，你**可以直接指向你自定义的**黑盒搜索 API**** ！

###### **RRE 企业能够直接评估你的搜索 API(与 Elasticsearch 或 Solr 进行内部对话)**

让我们看看如何进行配置:

首先，定义**黑盒搜索 API** 的端点:



![](img/145fdbb6de6edc3aa7045bb54bbe6606.png)

**N.B.** your ratings need to specify the full HTTP **Black Box Search API** request associated with the query of the <query, document, rating> triplet

然后您需要配置端点来访问搜索引擎实例本身( **Elasticsearch** 或 **Solr** ):



![](img/c12cbef0a326a646c3601810b13eb57f.png)

在这个例子中，我们正在配置一个 **Elasticsearch** 实例。
您可能会注意到一个不寻常的配置:“日志提取器端口”。
这是一个简单的脚本，您需要将其安装在托管 **Elasticsearch** 实例的环境中。这是免费的，如果您需要安装帮助，请告诉我们！

**视频:目标搜索引擎 4**

就是这样！现在你可以设置你的目标搜索引擎并继续下一集:
[RRE-企业:如何管理你的数据收集](https://web.archive.org/web/20221202232314/http://RRE-Enterprise: How to Manage Your Data Collections)

[https://web.archive.org/web/20221202232314if_/https://www.youtube.com/embed/YCsIbTDQCtU](https://web.archive.org/web/20221202232314if_/https://www.youtube.com/embed/YCsIbTDQCtU)

// begin your journey into the search quality evaluation

## 评级评估企业

[DISCOVER MORE AND DOWNLOAD](https://web.archive.org/web/20221202232314/https://sease.io/rated-ranking-evaluator-enterprise)// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Drop constant features:一个现实世界的学习排名场景的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！