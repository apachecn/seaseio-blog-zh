# Apache Solr:链接 SearchHandler 实例:CompositeRequestHandler

> 原文：<https://web.archive.org/web/sease.io/2018/03/compositesearchhandler.html>

## 什么是“隐形查询”？

这是格兰特·英格索尔关于 Lucidworks.com 的一篇文章的节选，文章谈到了无形的查询:

*“对于任何给定的用户查询，在许多应用程序中经常需要执行多个查询。例如，在要求非常高精度的应用中(只要求好的结果，忽略边缘结果)，应用程序。可能有几个字段，一个用于精确匹配，一个用于不区分大小写的匹配，还有一个用于词干匹配。给定一个用户查询，应用程序可能会首先尝试对精确匹配字段进行查询，如果有结果，则只返回该集合。如果没有结果，应用程序将继续搜索下一个字段，以此类推。”*

 *上面的句子假设了一个场景，其中(客户端)应用程序在一个用户查询之上向 Solr 发出几个后续请求(即**一个**用户查询= > **多个**搜索引擎查询)。那你没有这样的控制力呢？假设您是一个使用 Magento 构建的电子商务门户的搜索工程师，在这个场景中，Magento 充当 Solr 客户端；有人安装并配置了 Solr 连接器，一切正常:当用户提交搜索时，连接器将请求转发给 Solr，Solr 根据配置执行(单个)查询。

## 背景

现在，假设上面的查询没有返回任何结果。整个请求/响应交互都消失了，用户将会看到类似“对不起，没有您的搜索结果”的内容。虽然这听起来非常合理，但在本帖中，我们将关注一种不同的方法，基于你可以在上面的摘录中读到的“不可见查询”的东西。这里的要点是一个前提条件:我不能改变客户端代码；那是因为(例如):

*   *   我不想在我的 Magento / Drupal 实例中引入定制代码
    *   我不懂 PHP
    *   我严格负责搜索基础设施，前端开发人员不希望/不能够在客户端正确实现这一功能
    *   我想尽可能多地移动 Solr 中的搜索逻辑

What I’d like to do is to provide a single entry point (i.e. one single request handler) to my clients, being able to execute a workflow like this:

![Invisible Queries Apache Solr](img/7932d88389e3b22d4d53748c51fd9acb.png)

## 复合请求处理程序

潜在的想法是提供一个*门面*，它能够链接几个处理程序；大概是这样的:

```
<requestHandler name="/search" class="...CompositeRequestHandler">
    <str name="chain">/rh1,/rh2,/rh3</str>
</requestHandler> 
```

where */rh1*, */rh2* and */rh3* are standard *SearchHandler* instances you’ve already declared, that you want to chain in the workflow described in the diagram above.

*CompositeRequestHandler* 的实现实际上很简单:它的 *handleRequestBody* 方法将依次执行已配置的处理程序引用，并在收到第一个肯定的查询响应(通常是 numFound > 0 的查询响应，但该组件的最新版本允许您配置其他谓词)后中断链。逻辑大概是这样的:

chain.stream()// Get the request handler associated with a given name.map(refName -> requestHandler(request, refName))// Only SearchHandler instances are allowed in the chain.filter(SearchHandler.class::isInstance)// executes the handler logic.map(handler -> executeQuery(request, response, params, handler)).filter(qresponse -> howManyFound(qresponse) > 0)// Stop the iteration when the first condition above has been satisfied.findFirst()// or, if we don’t have any positive executions, just returns an empty response..orElse(emptyResponse(request, response)));

您可以在我们的 Sease GitHub 资源库[【2】](https://web.archive.org/web/20220929234549/https://github.com/SeaseLtd/composite-request-handler)中找到 CompositeRequestHandler 的源代码。像往常一样，我们热烈欢迎任何反馈。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929234549/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929234549/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929234549/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于链接 SearchHandler 实例的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！*