# Apache Solr ChildDocTransformerFactory:如何构建复杂的 ChildFilter 查询

> 原文：<https://web.archive.org/web/sease.io/2019/06/apache-solr-childfilter-transformer.html>

当使用嵌套文档和 Apache Solr 块连接功能时，通常需要查询一个实体(例如父实体)，然后为每个搜索结果检索所有(或部分)相关的子实体。



让我们看看这种功能最重要的方面，以及在检索搜索结果的子结果时如何应用复杂的查询。

## 如何索引嵌套文档

如果我们以 Json 格式提供文档，语法是非常直观的:

{
"id": "A "、
"queryGroup": "group1 "、
" _ child documents _ ":[
{
" metric score ":" 0.86 "、
"metric": "p "、
"docType": "child "、
"id": 12894
}、
{
"metricScore": "0.62 "、【T15

子文档以 Json 节点数组的形式传递，每个节点都有一个特定的 Id
**注意:如果您依赖 Apache Solr 使用 UUIDUpdateProcessorFactory[【1】](https://web.archive.org/web/20220929233516/https://lucene.apache.org/solr/8_0_0//solr-core/org/apache/solr/update/processor/UUIDUpdateProcessorFactory.html)为您分配 Id，这还不能用于子文档。在这种情况下，你应该实现你自己的更新请求处理器，它遍历所有的子进程，并给每个子进程分配一个 id(然后贡献给社区)🙂)**

 **如果您正在使用 SolrJ，并且计划通过代码索引和检索子文档，那么情况会稍微困难一些。
首先，让我们对 POJO 进行适当的注释:**  **公共类父类
{
@字段
私有字符串 id；
…

@ Field(Child = true)
私有列表<Child>children；

**N.B.** *Parent* ， *Child* 和*Child*都只是虚构的名字，这里重要的注释是 SolrJ annotation**@ Field(Child = true)**，你可以为你的 POJO 类和变量使用任何你喜欢的名字

###### SolrJ 中嵌套文档的索引

在编制索引时，您有两种选择，您可以使用文档活页夹:

DocumentObjectBinder Solr binder = new DocumentObjectBinder()；
Parent sample Parent = new Parent()；
Child sample Child = new Child()；

SolrInputDocument parent = binder . tosolrinputdocument(sample parent)；
SolrInputDocument child = binder . tosolrinputdocument(sample child)；
parent . addchilddocument(child)；

solr.add("收藏"，父)

或者您可以使用普通的 POJO:

Parent sampleParent = new Parent()。
Child sample Child = new Child()；

//需要在你的 POJO
sample parent . addchilddocument(sample child)中实现；

solr.addBean("集合"，sampleParent)

## 如何查询和检索嵌套文档

好了，我们讨论了索引方面，这并不简单，但是在这一点上，我们应该在索引中有嵌套的文档，很好地放在与父文档相邻的块中，以允许在查询时快速检索。
首先，让我们看看如何查询父母/子女并获得适当的响应。

###### 查询子代并检索父代

q={！parent which =<allparents>} <somechildren>例如

q={！parent which = docType:" parent " }标题:(子标题术语)</somechildren></allparents>

**n . b .***all parents*是一个匹配所有父项的查询，如果您稍后想要过滤一些父项，您可以使用过滤查询或一些附加子句:

如
*q =****+title:join****+{！parent which = "****content _ type:parent document****" }备注:SolrCloud*

子查询必须总是只返回**的**子文档。

###### 查询父代并检索子代

q={！ <allparents>} <someparents>之子如

q={！名为 lucene 的子代</someparents></allparents>

**注意**参数`*allParents*`是只匹配**父文档**的过滤器；在这里，您可以定义用于标识**所有父文档**的字段和值。
参数`*someParents*`标识将匹配一些父文档的查询。输出的是孩子。

###### 如何独立于查询检索子项

如果您有一个返回父级的查询，不管它是块连接查询还是普通查询，您可能也会对检索子文档感兴趣。
这可以通过子变压器[【2】](https://web.archive.org/web/20220929233516/https://lucene.apache.org/solr/guide/8_0/transforming-result-documents.html#child-childdoctransformerfactory)实现。

**【子代】–ChildDocTransformerFactory**

`fl=id,[child parentFilter=doc_type:book childFilter=doc_type:chapter]`

当使用这个转换器时，必须指定`parentFilter`参数*，除非*模式声明了`_nest_path_`。它与所有块连接查询的工作方式相同。其他可选参数包括:

`**childFilter**`:过滤应该包含哪些子文档的查询。当您有多层次的文档时，这尤其有用。默认为所有子级。这个查询支持一个特殊的语法来匹配嵌套的文档模式，只要在模式中定义了`_nest_path_`，并且查询在第一个`:`之前包含一个`/`。例子:`childFilter=/comments/content:recipe`

`**limit**`:每张父单据返回的子单据的最大数量。默认为`10`

【T10

`**fl**`:变压器要返回的字段列表。默认为顶级`fl`)。
还有一个限制，这里的字段应该是顶层`fl`参数指定的字段的子集。

**复杂子过滤器查询**

让我们关注一下 ***childFilter*** 查询。
该查询必须只匹配子文档。然后，只检索子文档的一个特定子集就可以了。
不幸的是，在这里传递复杂的查询不如预期的直观，因为默认情况下**空格会对你不利**。

*… childFilter=field:(经典或布尔 AND 查询)】*

*… childFilter=field:我是复杂查询】*

你当然可以在文本分析中尝试复杂的方法并调试解析后的查询，但是我推荐使用**局部参数占位符和替换**，这将解决你的大部分问题:

fl=id，[child parent filter = doc _ type:book child filter = $ child query limit = 100]
&child query =(field:(I 为复杂子查询或布尔型))

使用占位符替换将解决空白局部参数分割问题，并帮助您制定复杂的查询，以便仅从父结果中检索子文档的子集。

###### 在 SolrJ 中检索子文档

一旦有了返回子文档(可能还有父文档)的查询，让我们看看如何在 SolrJ 中使用它来取回 Java 对象。

DocumentObjectBinder Solr binder = new DocumentObjectBinder()；
String fields="id，query，"+
"[child parent filter = docType:parent child filter = $ child query]"；
String child query = " child field:value "；
最终 SolrQuery 查询=新 Solr QUERY(GET _ ALL _ PARENTS _ QUERY)；
query.add("metricFilter "，metric filter)；
query . addfilterquery(" parent field:value ")；
…
query.setFields(字段)；

query response children = Solr . query(" collection "，查询)；
List<Parent>parents = binder . get beans(Parent . class，children . get results())；

这样，您将获得满足查询的父对象，包括所有请求的字段和嵌套的子对象。

## 结论

使用嵌套文档非常有趣，可以解决很多问题和棘手的用户需求，但是它们也不容易掌握，所以我希望这篇博客可以帮助您在 Apache Solr 的块连接和嵌套文档的波涛汹涌的大海中航行！

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929233516/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929233516/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929233516/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于如何构建复杂的 ChildFilter 查询的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**