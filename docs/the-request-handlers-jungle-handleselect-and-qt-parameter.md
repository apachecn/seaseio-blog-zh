# 请求处理程序 Jungle–handle select 和 qt 参数

> 原文：<https://web.archive.org/web/sease.io/2019/12/the-request-handlers-jungle-handleselect-and-qt-parameter.html>

这篇博文的目标是阐明请求处理程序的主题以及它们与最近 Apache Solr ( >= 7.0)中 QT 请求参数的关系。

## 定义

用名称和类定义了一个**请求处理程序**。
它定义了一个 API 端点来处理 http 请求。
请求处理程序的名称在 Apache Solr 的请求中被引用，通常作为一个路径。例如，如果 Solr 安装在`[http://localhost:8983/solr/](https://web.archive.org/web/20220929230728/http://localhost:8983/solr/)`处，并且您有一个名为“gettingstarted”的集合，那么您可以发出如下请求:

```
http://localhost:8983/solr/gettingstarted/select?q=solr
```

这个查询将由名为`/select`的请求处理器处理。

Solrconfig.xml 中定义了请求处理程序，它们看起来像:

```
<requestHandler name="/select" class="solr.SearchHandler">
  <lst name="defaults">
    <str name="echoParams">explicit</str>
    <int name="rows">10</int>
  </lst>
</requestHandler>
```

在这篇博文中，我们将重点讨论搜索处理程序:一类处理搜索请求的请求处理程序。

**注意**这超出了这篇博文的范围，但是请求处理程序可以是一个复杂的组件，它包含高级逻辑，并且透明地向用户搜索请求添加/默认各种请求参数【T21【1】

**qt** 参数是 Apache Solr 请求调度程序支持的请求参数。它最初的作用域**是**通过一个请求参数来传递请求处理程序。
该参数与 Solr 请求调度程序的 handleSelect 属性紧密相关:

*第一个可配置项是`<requestDispatcher>`元素本身的`handleSelect`属性。该属性可以设置为两个值之一，即“真”或“假”。它管理 Solr 如何响应诸如`/select?qt=XXX`之类的请求。如果 requestHandler 没有用名称`/select`显式注册，默认值“false”将忽略对`/select`的请求。**值“真”将把查询请求路由到用`qt`值**定义的解析器。*
*在 Solr 的最新版本中，默认情况下会定义一个`/select` requestHandler，因此值为“false”也可以。* 阿帕奇 Solr Wiki

我们现在已经澄清了博客文章的定义和范围，让我们看看这是否总是正确的，以及在 SolrJ 中发生了什么。

## 关于 handleSelect 和 QT 的更多信息

如果 handleSelect 为 TRUE，并且请求中没有传递 QT 参数，Solr 会将/select 请求路由到 Solrconfig.xml 中配置的默认请求处理程序:

```
<requestHandler name="/myRequestHandler" class="solr.SearchHandler" default="true">
```

这个语句怎么样:
"***" true "值将查询请求路由到用`qt`值*** 定义的解析器。"上述陈述部分属实。
将应用 QT 参数，将请求路由到指定的请求处理程序

```
<requestDispatcher handleSelect="true">
        ...
    </requestDispatcher> 
```

**并且没有定义/select 请求处理程序**(注意，除非您删除它，否则默认定义/select 请求处理程序)。

在所有其他情况下，**QT 参数将被忽略**

## 索尔杰发生了什么

在 SolrJ 中，可以指定要使用的请求处理程序，调用这个 setter 方法:

```
SolrQuery mySolrQuery = new SolrQuery("userId:" + userId);
mySolrQuery.setRequestHandler("/myRequestHandler");
```

而且效果很好。

嗯，部分是…因为如果你不设置请求处理程序，SolrJ 将默认使用 **/select** 请求处理程序！
因此，如果您还定义了/select，那么您的实例的 Solrconfig.xml 中配置的默认请求处理程序将被忽略。
原因在 Solrclient 中是硬编码的:

```
org.apache.solr.client.solrj.impl.HttpSolrClient
...
/**
 * A SolrClient implementation that talks directly to a Solr server via HTTP
 */
public class HttpSolrClient extends BaseHttpSolrClient {

  private static final String UTF_8 = StandardCharsets.UTF_8.name();
  private static final String DEFAULT_PATH = "/select";
...

```

我们现在很好奇，那么 mysolrquery . setrequesthandler("/myRequestHandler ")是做什么的呢？

```
public SolrQuery setRequestHandler(String qt) {
    this.set(CommonParams.QT, qt);
    return this;
  }

```

哦不！它使用了 QT 参数！我们已经看到这是多么棘手！但是如果你在 SolrJ 中这样做的话，一切都很好，不考虑我们在文章开始时提到的关于手柄的考虑因素

T31

要理解为什么，我们可以看一下源代码，但是我们有一个提示，埋藏在 Apache Solr 文档中[【2】](https://web.archive.org/web/20220929230728/https://lucene.apache.org/solr/guide/8_3/major-changes-in-solr-7.html):

如果`luceneMatchVersion`是 7.0.0 或更高版本，那么`solrconfig.xml`中的`handleSelect`参数现在默认为`false`。这导致 Solr 忽略`qt`参数，如果它出现在请求中的话。如果您有不带前导“/”的请求处理程序，您可以设置`handleSelect="true"`或者考虑迁移您的配置。**`qt`参数仍然被用作 SolrJ 特殊参数，指定请求处理程序(尾部 URL 路径)使用**。

Solr Wiki

因此，似乎知道 SolrJ 使用参数不同…

但是怎么做呢？

这段代码最终澄清了一切:

```
org.apache.solr.client.solrj.request.QueryRequest#getPath
public String getPath() {
    String qt = query == null ? null : query.get( CommonParams.QT );
    if( qt == null ) {
      qt = super.getPath();
    }
    if( qt != null && qt.startsWith( "/" ) ) {
      return qt;
    }
    return "/select";
  }
```

由 SolrClient 自己调用:

```
org.apache.solr.client.solrj.impl.HttpSolrClient#createMethod
protected HttpRequestBase createMethod(SolrRequest request, String collection) throws IOException, SolrServerException {
   ...
    String path = requestWriter.getPath(request);
    if (path == null || !path.startsWith("/")) {
      path = DEFAULT_PATH;
    }  
```

希望这能帮助一些人！

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929230728/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929230728/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929230728/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 handleSelect 和 qt 参数的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！