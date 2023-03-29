# Apache Solr 中如何使用特征向量缓存

> 原文：<https://web.archive.org/web/sease.io/2022/02/how-the-feature-vector-cache-is-used-in-solr.html>

大家好！在这篇博文中，我想谈谈 Solr 中特征向量缓存的当前使用情况。

你可以在这篇[博文](https://web.archive.org/web/20220925181559/https://sease.io/2021/11/feature-vector-cache-in-solr.html)中找到学习 Solr 排名的简要介绍！



(这篇博文是看着 Apache Solr 9.0 中的代码和 Solr git [库](https://web.archive.org/web/20220925181559/https://github.com/apache/solr)主分支中的 commit id=cfc953b6b90 编写的。)

## TL；DR；

目前，特征向量缓存仅在 fl 参数中启用特征转换器时使用**(在插入和查找中)。
独立于特征转换器，在重新排序时也使用特征向量缓存将是有趣的。**

我们正计划捐款。

## LTR 查询性能

###### 特征向量缓存

阅读 Apache Solr 文档，特征向量缓存的好处并不太直观。
让我们直接从 Solr 代码来看看它是如何工作的。

**插入和查找在一个名为*org . Apache . Solr . ltr . feature logger*的特定类中完成**

**插入**在*org . Apache . Solr . ltr . feature logger # log*方法中完成:

```
public boolean log(int docid, LTRScoringQuery scoringQuery, SolrIndexSearcher 
searcher, LTRScoringQuery.FeatureInfo[] featuresInfo) {
  final String featureVector = makeFeatureVector(featuresInfo);
  if (featureVector == null) {
    return false;
  }
  return searcher.cacheInsert(fvCacheName, fvCacheKey(scoringQuery, docid), 
featureVector) != null;
}
```

**查找**在*org . Apache . Solr . ltr . feature logger # getFeatureVector*方法中完成:

```
public String getFeatureVector(int docid, LTRScoringQuery scoringQuery, 
SolrIndexSearcher searcher) {
    return (String) searcher.cacheLookup(fvCacheName, fvCacheKey(scoringQuery, 
docid));
}
```

**注意:只有在使用特征转换器(在 fl 查询参数中)时，插入和查找才会发生。**

###### 插入

让我们深入了解一下插入过程。

**为了有一个插入:**

我们遇到的**第一个条件**在*org . Apache . Solr . ltr . ltr recorder # score features*方法中:

```
org.apache.solr.ltr.LTRRescorer#scoreFeatures

....
if (scoreSingleHit(topN, docBase, hitUpto, hit, docID, scorer, reranked)) {
  logSingleHit(indexSearcher, modelWeight, hit.doc, scoringQuery);
}
...
```

**目前,*org . Apache . Solr . ltr . ltr recorder # scores ingleit*方法上的条件总是被验证。我们怀疑这是由于代码维护问题导致的，我们正在对此进行更多的调查，稍后我们会更新这篇文章的更多细节。**

我们来看看为什么。

下面是被调用的方法:

```
org.apache.solr.ltr.LTRRescorer#scoreSingleHit

...
if (hitUpto < topN) {
      reranked[hitUpto] = hit;
      // if the heap is not full, maybe I want to log the features for this
      // document
      logHit = true;
    } else if (hitUpto == topN) {
      // collected topN document, I create the heap
      heapify(reranked, topN);
    }
...
```

这段代码在 topN 文档的重排序阶段被调用(该参数对应于查询时在 *rq* 参数中选择的*重排序文档*值)。
因为我们正在对 topN 文档进行重新排序，所以对于这些 topN 文档，if 子句 *hitUpTo < topN* 总是被验证(我们正在对它们进行迭代),因此对于它们，变量 *logHit* 总是被设置为 *True* 。

然后第二个条件出现。

在*org . Apache . Solr . ltr . ltr recorder # logSingleHit*中，需要有一个 *FeatureLogger* 以及一个 *SolrIndexSearcher* 。

条件是这样的:

```
org.apache.solr.ltr.LTRRescorer#logSingleHit

...
if (featureLogger != null && indexSearcher instanceof SolrIndexSearcher) {
  featureLogger.log(docid, scoringQuery, (SolrIndexSearcher)indexSearcher, 
modelWeight.getFeaturesInfo());
}
...
```

**拥有 *SolrIndexSearcher* 的条件总是满足的，所以让我们把重点放在 *FeatureLogger* 上。**

如果 1 和 2 都满足，则设置一个*特征记录器*:

1.  1.  使用 *fl* 查询参数中的特征转换器。
        在这种情况下，我们将从第 164 行把*org . Apache . Solr . ltr . search . ltrqparserplugin . ltrqparser # parse*方法中的 *extractFeatures* 变量设置为 *True* 。
    2.  *特征转换器*中定义的特征库与或
        模型中定义的特征库相同，只是设置了特征转换器*【features】*组件，没有指定我们想要使用的库。

**因此，要进行插入，需要满足两个条件:**

1.  1.  使用 *fl* 查询参数**中的**特征转换器**。**
    2.  *fl* 参数**中定义的**特征库**与
        *或*** 中定义的特征库相同，只是**特征转换器***组件**已设置**，**未指定***

 *###### **只给好奇的人**

###### T95

如果您想深入了解这些条件，可以看看 o*rg . Apache . Solr . ltr . search . ltrxparserplugin . ltrxparser # parse*方法。
此处 *extractFeatures* 变量设置在第 164 行:

```
final boolean extractFeatures = SolrQueryRequestContextUtils.isExtractingFeatures(req);
```

然后，在第 183 行中使用计算出的 *extractFeatures* ，其中实现了特征存储的条件:

```
final boolean featuresRequestedFromSameStore = 
(modelFeatureStoreName.equals(tranformerFeatureStoreName) || 
tranformerFeatureStoreName == null) ? extractFeatures : false;
```

然后计算出的*featuresRequestedFromSameStore*用于在第 198 行设置*特征记录器*。

###### 查找

如前所述，查找是转换过程的一部分。
要启用转换器，只需在 fl 参数中传递<转换器名称>(在 solrconfig.xml 中定义)。
**例如:fl=【特性】**

solrconfig.xml

```
<transformer name="features" 
class="org.apache.solr.ltr.response.transform.LTRFeatureLoggerTransformerFactory">
  <str name="fvCacheName">QUERY_DOC_FV</str>
</transformer>
```

那么**这两个条件**中的一个需要*为真*:

1.  1.  查询不必是 OriginalRankingLTRScoringQuery。
    2.  需要在 *fl* 参数中明确定义一个商店。



以下是需要验证的两个条件:

```
org.apache.solr.ltr.response.transform.LTRFeatureLoggerTransformerFactory.FeatureTra
nsformer#implTransform

...
if (!(rerankingQuery instanceof OriginalRankingLTRScoringQuery) || 
hasExplicitFeatureStore) {
        Object featureVector = featureLogger.getFeatureVector(docid, rerankingQuery, 
searcher);
        ...
      }
...
```

**当交叉未运行**时，*originalrankingtrscoringquery*的第一个条件始终为真，因此我们将始终访问缓存进行查找。
交错为您提供了将学习模型与原始 Solr 分数进行比较的可能性。
原始 Solr 分数不需要提取特征，所以除非定义了 hasExplicitFeatureStore，否则没有必要进行查找。

这里我们可以看到，缓存仅用于*feature logger transformer*，而不是在通过 *rq* 参数完成的重新排序阶段。我们没有加快重新排序的速度。

**因此，要进行查找，需要满足两个条件:**

T42

1.  1.  使用 *fl* 查询参数**中的**特征转换器**。**
    2.  **LTR 模型是一个学习过的模型(而不是原始的 Solr 分数预分级)
        *或*** 一个**特征库**已经被 **exp**

###### 例子

让我们做一些示例查询来看看 Solr 缓存是如何工作的。

###### 第一次查询

![](img/ca3cf6c56f39a074d002c8d57ffb069b.png)

这是我们的第一个查询。在 *fl* 查询参数中定义了【T65【特性】组件，并且已经使用了 *rq* 查询参数。

对于插入，我们可以看到:

| 情况 | 条件得到验证了吗 |
| --- | --- |
| *变压器*在 fl ('[features=…]')中定义 | **是** |
| 定义的 **变压器** *商店*与模型商店【真】
或
相同*变压器*【特性】已设置，未指定要使用的商店(假) | **是** |

**插入的所有条件都满足，因此插入发生。**

对于查找，我们可以看到:

| 情况 | 条件得到验证了吗 |
| --- | --- |
| *变压器*在 fl ('[features=…]')中定义 | **是** |
| 该模型是一个学习模型(' first_model ')(真)
或
在*转换器* ('first_model_store ')(真)中定义了一个显式存储 | **是** |

**查找的所有条件都满足，因此发生查找。**

T31

###### 第二次查询

![](img/ccfcbabf86a6eff65ac10f57e398bcb3.png)

这是我们的第二个查询。在 *fl* 查询参数中定义了*【特性】*组件，并且 *rq* 查询参数也被使用。

对于插入，我们可以看到:

| 情况 | 条件得到验证了吗 |
| --- | --- |
| *变压器*在 fl ('[features=…]')中定义 | **是** |
| 定义的 **变压器** *商店*与模型商店【假】
或
相同*变压器*【特性】已设置，未指定要使用的商店(假) | **否** (模特店是‘first _ model _ store’) |

**并非所有的插入条件都满足，因此不会进行插入。**

T73

对于查找，我们可以看到:

| 情况 | 条件得到验证了吗 |
| --- | --- |
| *变压器*在 fl ('[features=…]')中定义 | **是** |
| 该模型是一个学习模型(' first_model') (TRUE)
或
在*转换器*(' second _ model _ store ')(TRUE)中定义了一个显式存储 | **是**
(注意型号和变压器店不同) |

**查找的所有条件都满足，因此发生查找。**

###### 第三个查询



![](img/70c329f9ae040675f510f9229041ef58.png)

这是我们的第三个查询。在 *fl* 查询参数中定义了*【特性】*组件，但是没有使用 *rq* 查询参数。

对于插入，我们可以看到:

| 情况 | 条件得到验证了吗 |
| --- | --- |
| *变压器*在 fl ('[features=…]')中定义 | **是** |
| 定义的 **变压器** *商店*与模型商店【假】
或
相同*变压器*【特性】已设置，未指定要使用的商店(假) | **没有** (没有 rq 参数) |

**并非所有插入条件都满足，因此不会进行插入。**

T53

对于查找，我们可以看到:

| 情况 | 条件得到验证了吗 |
| --- | --- |
| *变压器*在 fl ('[features=…]')中定义 | 是 |
| 该模型是一个学习模型(假)
或
一个显式存储已在*转换器*(‘second _ model _ store’)中定义(真) | 是 |

**查找的第一个和第二个条件被满足，因此查找发生。**

###### 第四个查询



![](img/7956622c0bdba101214ced9d9f20386b.png)

这是我们的第四个查询。在 *fl* 查询参数中没有定义【T76【特性】组件，但是 *rq* 查询参数已经被使用。

T83

对于插入，我们可以看到:

| 情况 | 条件得到验证了吗 |
| --- | --- |
| *变压器*在 fl ('[features=…]')中定义 | **没有** |
| 定义的 **变形金刚** *店铺*与模型店铺
或
相同*变形金刚*【特性】已经设置，没有指定要使用的店铺(FALSE) | **没有** |

**并非所有的插入条件都满足，因此不会进行插入。**

对于查找，我们可以看到

| 情况 | 条件得到验证了吗 |
| --- | --- |
| 在 fl ('[features=…]')中没有定义*变压器* | **没有** |
| 该模型是一个学习模型(真)
或
一个显式存储已在*转换器*中定义(假) | **是** |

****不是所有的查找条件都满足，因此不会进行查找。T47****

## 未来作品

我们将很快打开一个吉拉问题，在重新排序阶段集成特征向量缓存，独立于特征转换器。

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做[学习排名](https://web.archive.org/web/20220925181559/https://sease.io/learning-to-rank-training)和 [Apache Solr 初学者](https://web.archive.org/web/20220925181559/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)培训吗？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220925181559/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 如何使用特征向量缓存的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！*