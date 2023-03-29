# 使用标记和排除的 Apache Solr 方面和 ACL 过滤器

> 原文：<https://web.archive.org/web/sease.io/2018/12/apache-solr-facets-and-acl-filters-using-tag-and-exclusion.html>

当结果中的文档被访问控制列表过滤时，字段上的方面聚合会发生什么情况？在这种情况下，使用 facet mincount 参数很重要。
指定结果集中出现在响应中的方面值的最小计数:

*   *   **mincount=0** ，在响应中返回语料库中存在的所有方面值。这包括与已被 ACL 过滤掉的文档相关的内容(0 计数面)。这可能会导致一些不好的副作用:比如用户看到了他/她不应该看到的方面值(因为 ACL 从结果集中过滤掉了那个文档)。
    *   **mincount=1** ，只返回与结果集中至少一个文档匹配的方面值。这种配置是安全的，用户将只能看到由 ACL 控制的方面值。他们实际上只会看到他们应该看到的东西。



但是，如果您希望看到 0 计算方面值，但保留 ACL，会发生什么情况呢？
这可能有助于您更好地理解值在整个语料库中的分布，但是 ACL 仍然有效，因此用户仍然只能看到他们应该看到的可能值。
标签和排除在这种情况下很方便。

## 刻面标记和排除

标记和排除是 Apache Solr 中一个非常重要的特性，你不会相信它被误用或完全忽略了多少次，给用户带来了不稳定的体验。
让我们看看它是如何工作的:

###### 磨尖

您可以使用 Solr 本地参数语法标记过滤器查询:

```
fq={!tag=docTypeFilter}doctype:pdf
```

这同样适用于主查询(如果您使用显式查询解析器，则有一些警告) :

```
q={!tag=mainQuery}I am the main query

q={!edismax qf=text title tag=mainQuery}I am the main query
```

当分配一个标签时，我们给 Solr 单独识别各种搜索子句(例如主查询或过滤查询)的可能性。
实际上，这是一种为搜索查询或过滤器分配标识符的方式。

###### 排除在传统刻面中

当应用过滤器查询时，Solr 减少了结果空间，消除了不满足附加过滤器的文档。
让我们假设我们想要计算结果集上一个方面的值，忽略由过滤查询添加的额外过滤。
实际上可以等效于在应用减少结果集的过滤器之前对结果集状态的方面值进行计数的概念。Apache Solr 允许您这样做，而不会影响最终返回的结果。

这被称为排斥，可以逐个方面地应用。

```
fq={!tag=docTypeFilter}doctype:pdf...&facet=true&
facet.field={!ex=docTypeFilter}doctype
```

这将计算结果集上的“doctype”字段方面，不包括带标记的筛选器(因此，在计算此类聚合时，将不应用“doctype:pdf”筛选器，而将在扩展的结果集上计算计数)。所有其他方面、聚合和结果集本身都不会受到影响。

```
1.<Wanted Behaviour - applying tag and exclusion>
=== Document Type === 
[ ] Word (42) 
[x] PDF  (96) 
[ ] Excel(11) 
[ ] HTML (63)
```

这对于单值字段尤其有用:
当选择一个方面值并刷新搜索时，如果不应用标记和排除，您将只在方面中获得该值，从而破坏该字段的细化和探索方面功能。

```
2.<Unwanted Behaviour - out of the box>
=== Document Type === 
[ ] Word (0) 
[x] PDF  (96) 
[ ] Excel(0) 
[ ] HTML (0)
```

```
3.<Unwanted Behaviour - mincount=1>
=== Document Type === 
[x] PDF  (96) 
```

正如你在 2 中看到的。第三。方面变得几乎不可用于进一步探索结果，这可能会使用户体验被大量来回选择和取消选择过滤器的活动所分割。

###### 在 Json Faceting 中排除

在标记过滤器之后，用 json.facet 方法应用排除非常简单:

```
visibleValues: {
     type: terms,
     field: cat,
     mincount: 1,
     limit: 100,
     domain: {
         excludeTags: <tag>
     }
 }
```

定义 json 方面时，应用排除只是添加定义了 excludeTags 的域节点。

## 标记和排除以在 0 次计数中保留 Acl 过滤

###### 问题

*   *   用户受制于一组限制其结果可见性 ACL。
    *   他们还希望看到 0 计数方面，以便更好地理解结果集和语料库。
    *   您不想使 ACL 控制无效，所以您不希望它们看到合理的方面值。

###### 标记主查询和 Json Faceting

这是通过 Json faceting 的标记和排除的组合实现的。
首先，我们要标记主查询。
我们假设 ACL 控制将是一个过滤查询(我们建议使用经过适当调整的过滤查询来应用 ACL 过滤)。
标记主查询并将其从方面计算中排除将允许我们获得 ACL 过滤语料库中的所有方面值(主查询将被排除，但 ACL 过滤查询仍将被应用)。

```
q={!edismax tag=mainQuery qf=name}query&fq=aclField:user1...
json.facet={visibleValues: {
     type: terms,
     field: cat,
     mincount: 1,
     limit: 100,
     domain: {
         excludeTags: mainQuery
     }
 }}
```

我们就快到了，这个方面聚合将给出原始语料库(应用了 ACL)中对用户可见的所有方面值的计数。
但是我们想要的是基于当前结果集和所有可见的 0 计数方面的正确计数。
为此，我们可以向 Json 分面请求添加一个块:

```
q={!edismax tag=mainQuery qf=name}query&fq=aclField:user1...
json.facet={
     resultSetCounts: {
         type: terms,
         field: category,
         mincount: 1
     },
     visibleValues: {
         type: terms,
         field: category,
         mincount: 1,
         domain: {
             excludeTags: mainQuery
         }
     }
 }
```

*   *   **resultset counts**–是结果集中的计数，仅包括不为 0 的计数方面值。这是用户在具有正确计数的当前结果集上可见的值列表。
    *   **visible values**–用户应该可以看到结果集中的所有方面值吗

T12

然后，根据我们想要提供的用户体验，我们可以使用这些信息块来适当地呈现最终的响应。
例如，我们可能想要显示所有可见的值，并在可用时将结果集计数中的一个计数与它们相关联。

```
=== Document Type - Result Counts === 
[ ] Word (10) 
[ ] PDF  (7) 
[ ] Excel(5) 
[ ] HTML (2)
 === Document Type - Visible Values === 
[ ] Word (100) 
[ ] PDF  (75) 
[ ] Excel(54) 
[ ] HTML (34)
[ ] Jpeg (31) 
[ ] Mp4  (14)
* [ ] SecretDocType1 (0) -> not visible, mincount=1 in visibleValues** [ ] SecretDocType2 (0) -> not visible, mincount=1 in visibleValues*

=== Document Type - Final Result for users === 
[ ] Word (10) *-> count is replaced with effective result count* 
[ ] PDF  (7)  *-> count is replaced with effective result count* 
[ ] Excel(5)  *-> count is replaced with effective result count*
[ ] HTML (2)*-> count is replaced with effective result count*
[ ] Jpeg (+31) 
[ ] Mp4  (+14)
```

###### 额外收获:如果我在 Solrconfig.xml 中定义查询解析器会怎么样

如果您使用 solrconfig.xml 中定义的查询解析器，这个解决方案仍然有效。需要格外小心来标记主查询。
您可以使用 Solr 请求参数中的本地参数来实现:

```
*solrconfig.xml*
<lst name="defaults">
...
<str name="q">{!type=edismax tag=mainQuery v=$qq}</str>
<str name="qq">*:*</str>
...

*Query Time*
.../solr/techproducts/browse?qq=ipod mini&fq=acl:user1&json.facet=...
```

希望这有助于处理 ACL 或通用过滤器查询和刻面！

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930004226/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220930004226/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930004226/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 方面和 ACL 过滤器的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！