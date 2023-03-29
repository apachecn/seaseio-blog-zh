# 同义词+停用词？？OMG！

> 原文：<https://web.archive.org/web/sease.io/2018/07/combining-synonyms-and-stopwords.html>

## 背景

场景描述非常简单:我们想要使用**同义词**和**停用词**。

沿着我们上一篇[文章](https://web.archive.org/web/20220930004318/https://sease.io/2018/07/apache-solr-elasticsearch-auto-phrasing-out-of-the-box.html)的路径，我们将在分析链中引入一个额外的组件:一个 *StopFilter* ，顾名思义，它从传入的令牌流中删除一组单词。

我们将通过示例使用以下数据:

*   *   同义词= ["超出保修期"，" oow"]
    *   停用字词= ["of"]

可以在索引和/或查询时配置令牌过滤器。在这个上下文中，我们主要关注查询端:同义词和停用词都只在查询分析器中配置。

专门在查询时工作有一个很大的好处:我们可以在运行时改变事情，而不需要任何重新索引。同时，索引时不会执行非索引词过滤，因此这些词将成为字典中无用的部分。

## 问题是:同义词后面跟着停用词

我们有以下分析仪:

*   *   **指标分析器**
        *   标准符号化器
        *   小写字母
    *   **查询分析器**
        *   标准符号化器
        *   小写+ **同义词** + **停用词**

理论上，在查询分析器中，我们有两种选择:停用字词过滤器可以在同义词过滤器之前或之后定义。然而，第一种方法(before)没有多大意义，因为在同义词检测之前，作为停用词并且同时是同义词的一部分的术语将被删除。因此，不会检测到这些同义词:在示例数据中，发出如下查询

q =超出保修期

“of”术语将被*停止过滤器*删除，后续过滤器将收到[“out”、“warranty”]，这与配置的同义词(“out of warranty”)不匹配。

*Elasticsearch 用户:Elasticsearch 根本不允许这种场景；如果您尝试将 PUT Settings API 与上面定义的链一起使用(首先是停用词，然后是同义词与一些术语相交)，它将抛出一个非法参数异常，如" term:out of warranty analyzed to token(warranty)with position increment！= 1(got:2)。*

*Apache Solr 使用了一种宽松的方法:在创建索引时没有错误，但是问题仍然存在(我个人更喜欢弹性搜索方法)*

因此，显而易见的选择是在同义词过滤之后推迟停用词管理。不幸的是，这里有一个问题:停用词的移除在生成的令牌图中有一些不希望的副作用，并且查询解析器生成了一个错误的查询，因为它消耗了链末端的令牌流。

假设我们有以下查询:

q =电视坏了的**保修有事**的

 **它将生成以下内容:

标题:电视标题:去了(标题: **oow** **PhraseQuery(标题:** “出去了？保修 **有事****)**

正如您所看到的，同义词(超出保修范围-> oow)被正确地检测到，但是停用字词过滤器会删除所有“of”标记，即使第一个出现的单词是同义词的一部分。在生成的查询中，您可以看到偷偷摸摸的效果:通过删除第一个“of”出现创建的“洞”，在短语查询中包含了流中的下一个可用标记(在本例中为“something”)。

换句话说， **oow** 令牌同义词被标记为 positionLength = 3，这正确地意味着它跨越了三个令牌(1=out，2=of，3 = warranty)；稍后，查询解析器将包括用于生成同义词短语查询的下三个可用术语，但是由于我们不再具有第二个标记(of)，这样的计数还包括“something”，这是流中的第三个可用标记。

继续之前:这是一个已知的问题，是 Lucene 中一个长期存在的问题[【1】](https://web.archive.org/web/20220930004318/https://issues.apache.org/jira/browse/LUCENE-4065)，它有一个更广泛的领域，因为它与 *FilteringTokenFilter* ，即 *StopFilter* 的超类相关。

我们将试图解决的问题是:我们如何在查询时管理同义词和停用词，而不产生上面的冲突？

## 一个解决方案

首先要注意:我们将要创建的令牌过滤器只处理 Lucene 类。然而，当需要将东西插入运行时容器(例如 Apache Solr 或 Elasticsearch)时，部署过程取决于目标平台:我们在这里不讨论这一部分。

建议的解决方案是创建一个 *StopFilter 子类*，它将是“同义词感知的”；在决定是否需要从流中移除令牌之前，它将检查*令牌类型*和*位置长度*属性。目标是避免删除那些已经在停用词列表中定义但属于同义词定义的术语。

我们要扩展的类是*org . Apache . Lucene . analysis . core . stop flter*。这是一个空类，因为所有的过滤逻辑都在超类中(*org . Apache . Lucene . analysis . stop filter*和更通用的*org . Apache . Lucene . analysis . filteringtokenfilter*)。停用词逻辑驻留在 *accept()* 方法中，正如您所见，这非常简单:

```
protected boolean accept() {
  return !stopWords.contains(termAtt.buffer(), 0, termAtt.length());
}
```

如果停用字词列表包含当前术语，它将被删除。到目前为止，一切顺利。在调用上面的逻辑之前，我们需要扩展(实际上我们也可以修饰)用于做其他事情的*停止过滤器*类。

首先，我们需要检查**标记类型**:如果一个标记被标记为同义词**那么我们的过滤器不需要删除它。然后我们需要检查 **positionLength** 属性，因为在同义词检测上下文中，位置长度大于 1 意味着我们遍历了一个多术语同义词:**

```
public class SynonymAwareStopFilter extends StopFilter {

  private TypeAttribute tAtt = 
                             addAttribute(TypeAttribute.class);
  private PositionLengthAttribute plAtt = 
                             addAttribute(PositionLengthAttribute.class);

  private int synonymSpans;

  protected SynonymAwareStopFilter(
                         TokenStream in, CharArraySet stopwords) {
    super(in, stopwords);
  }

  @Override
  protected boolean accept() {
    if (isSynonymToken()) {
      synonymSpans = plAtt.getPositionLength() > 1 
                             ? plAtt.getPositionLength() 
                             : 0;
      return true;
    }

    return (--synonymSpans > 0) || super.accept();
  }

  private boolean isSynonymToken() {
    return "SYNONYM".equals(tAtt.type());
  }
```

让我们做些测试。我们将使用 Apache Solr 7.4.0 来检查结果。这里是字段类型定义，您可以看到我们的 SynonymAwareStopFilter:

```
<fieldtype name="text" class="solr.TextField" autoGeneratePhraseQueries="true">
       <analyzer type="index">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
       </analyzer>
       <analyzer type="query">
           <tokenizer class="solr.StandardTokenizerFactory"/>
           <filter class="solr.LowerCaseFilterFactory"/>
           <filter class="solr.SynonymGraphFilterFactory" 
                   synonyms="synonyms.txt" 
                   ignoreCase="false" 
                   expand="true"/>
           <filter class="sc.SynonymAwareStopFilterFactory" 
                   words="stopwords.txt" 
                   ignoreCase="true"/>
       </analyzer>
</fieldtype>
```

这是一个最小的请求处理器:

```
<requestHandler name="/def" class="solr.SearchHandler" default="true">
       <lst name="defaults">
           <bool name="sow">false</bool>
           <str name="df">title</str>
           <str name="defType">lucene</str>
           <bool name="debug">true</bool>
       </lst>
   </requestHandler>
```

运行上一个查询:

```
q=tv went out of warranty something of
```

我们有以下内容:

```
title:tv title:went (title:oow PhraseQuery(title:"out of warranty")) title:something
```

如果我们使用另一个同义词变体:

```
q=tv went oow something of
```

我们有以下内容:

```
title:tv title:went (PhraseQuery(title:"out of warranty") title:oow) title:something
```

一切似乎都在按预期运行！这可能只是 LUCENE-4065 所处理的场景中的一个特定场景；然而，它给了我很大的帮助，因为这(至少在我的经验中)是一个常见的用例。

像往常一样，我们热烈欢迎任何反馈。下次见！

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220930004318/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220930004318/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930004318/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于同义词+停用词的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**