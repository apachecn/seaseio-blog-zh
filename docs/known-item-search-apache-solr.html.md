# Apache Solr:编排已知项目和全文搜索

> 原文：<https://web.archive.org/web/sease.io/2018/05/known-item-search-apache-solr.html>

## 方案

你是 XYZ 有限公司的搜索工程师，该公司销售电子元件。XYZ 向您提供了过去六个月的申请日志，以及一些业务需求。

###### 两种客户，两种需求，两种搜索

日志分析表明，XYZ 主要有两类客户:第一类是“专家”用户(例如电工、经销商、商店)，其成员通过产品标识符、代码(例如 SKU、型号代码、像 Y-M8GB、140-213/A 和 ABD9881 这样的想法)来查询系统；很明显，至少看起来是这样，他们已经知道自己想要什么，寻找什么。但是，您会注意到很多这样的查询没有结果。经过调查，问题似乎是代码和标识符很难记住:查询使用许多不同的形式指向同一产品。例如:

*   *   y-m8gb(小写)
    *   YM8GB(无分隔符)
    *   YM-8GB(分隔符在错误的位置)
    *   Y/M8GB(错误的分隔符)
    *   Y M8GB(空白代替分隔符)
    *   y M8/gb(以上情况的组合)

在集合中只有一个相关文档的这种场景，通常被称为“已知项目搜索”:我们的第一个需求是确保这个*“产品标识符意图”*得到满足。

另一组客户是最终用户，就像我和你一样。由于不太熟悉代码或型号代码等产品规格，这里的行为有所不同:他们使用简单的关键字搜索，试图通过输入代表名称、品牌、制造商的术语来匹配产品。接下来是第二个需求，可以总结如下:人们必须能够通过输入普通的自由文本查询来找到产品。

可以想象，在这种情况下，搜索需求与其他场景不同:这里的重点是“以术语为中心”，因此涉及到我们需要应用的文本分析的不同考虑。

虽然专家组查询应该指向一个且只有一个产品(我们处于非黑即白的场景:匹配或不匹配)，但另一方面的需求要求系统根据输入的术语提供一个“相关”文档的列表。

*在继续之前，一个重要的事情/假设:为了说明的目的，我们将认为这两个查询/用户组是不相交的:也就是说，一个给定的用户只属于所提到的组中的一个，而不是两个。更好的是，给定的用户查询将包含产品标识符* ***或*** *术语，而不是两者都包含。*

## 模式和配置说明

###### 专家组和“已知物品搜索”

“product identifier”意图(假定隐含在该组的查询行为中)可以通过应用以下分析器在索引和查询时捕获，该分析器基本上将传入值视为一个整体，将其规范化为小写，删除所有分隔符，最后在单个输出标记中折叠所有内容。

```
<fieldtype name="identifier" class="solr.TextField" omitNorms="true">
    <analyzer>
        <tokenizer class="solr.KeywordTokenizerFactory" />
        <filter class="solr.LowerCaseFilterFactory" />
        <filter class="solr.WordDelimiterGraphFilterFactory"
                generateWordParts="0"
                generateNumberParts="0"
                catenateWords="0"
                catenateNumbers="0"
                catenateAll="1"
                splitOnCaseChange="0" />
    </analyzer>
</fieldtype>
<field name="product_id" type="identifier" indexed="true" ... />
```

在下表中，您可以通过一些示例看到分析器的运行情况:

![](img/c9bd28917f98644e7e4b2cea84538abe.png)

如您所见，分析器没有声明一个*类型的*属性，因为它应该在索引和查询时都被应用。但是，传入的值有所不同:在索引时，分析器处理的是字段内容(即传入文档的字段值)，而在查询时，流经管道的值由用户输入的一个或多个术语组成(简单地说，是一个查询)。

虽然在索引时一切都按预期工作，但在查询时，上面的分析器需要 Solr 6.5 中引入的一个特性:空格上的“**分割”标志[【1】](https://web.archive.org/web/20220925171301/https://lucidworks.com/2017/04/18/multi-word-synonyms-solr-adds-query-time-support)。当它被设置为“false”时(正如我们在这个上下文中所需要的)，它使得传入的查询文本在被发送到分析器时被保存为一个完整的单元。**

**在 Solr 6.5 之前，我们没有这样的控制，分析器接收一个“用空格预先标记”的标记；换句话说，查询时分析的工作单元是单个术语:分析器链(包括标记器本身)被调用来处理由前置空格标记化输出的每个术语。因此，我们的分析器在查询时不能像预期的那样工作:如果我们从上表中选择例子#5 和#6，您可以看到用户输入了一个空格。将“空白分割”标志设置为 true(显式地，或者使用存储在索引中的 Solr < 6.5), the pre-tokenization described above produces two tokens:**

 ***   *   #5 = {“Y”, ”M8GB”}
    *   #6 = {“y”, “M8/gb”}

so our analyzer would receive 2 tokens (for each case) and there won’t be any match with the single term **ym8gb** 。因此，在 Solr 6.5 之前，我们有两种方法来处理这个需求:

*   *   **客户端**:用双引号将整个查询括起来，用“\”转义空格，或者用像“-”这样的分隔符替换它们。很简单，但是它需要对客户端代码进行控制，而这并不总是可行的。
    *   **Solr side** :对传入的查询应用与上面相同的转换，但这次是在查询解析器级别。如果你了解一些 Lucene / Solr 的内部知识，这很容易。此外，它需要一个在 Solr 中安装定制插件的上下文。使用 UpdateRequestProcessor 也可以获得类似的效果，它将创建一个与原始字段值相同但没有任何空白的新字段。

###### 最终用户组和全文搜索查询

在这种情况下，我们在一个“普通的”全文搜索上下文中，其中分析确定了几个目标字段:产品名称和品牌。

与前面的场景不同，这里我们没有一个惟一的、确定的方法来满足搜索需求。这取决于很多因素:目录、术语分布、实现者体验、用户搜索体验方面的客户期望。所有这些都会导致不同的答案。举个例子，这里有一个可能的选择:

```
<fieldType name="brand" class="solr.TextField" omitNorms="true">
    <analyzer>
        <charFilter 
                class="solr.MappingCharFilterFactory" 
                mapping="mapping-FoldToASCII.txt"/>
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.StopFilterFactory" 
                ignoreCase="true" 
                words="lang/en/brand_stopwords.txt"/>
    </analyzer>
</fieldType>

<fieldType name="name" class="solr.TextField">
    <analyzer>
        <charFilter 
                  class="solr.MappingCharFilterFactory" 
                  mapping="mapping-FoldToASCII.txt"/>
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter 
                class="solr.StopFilterFactory" 
                ignoreCase="false" 
                words="lang/en/product_name_stopwords.txt"/>
        <filter class="solr.EnglishPossessiveFilterFactory"/>
        <filter class="solr.EnglishMinimalStemFilterFactory"/>
        <filter class="solr.RemoveDuplicatesTokenFilterFactory"/>
        <filter class="solr.LengthFilterFactory" min="2" max="50" />
    </analyzer>
</fieldType>
```

这里的重点不是模式设计本身:要强调的重要一点是，这个需求需要一个与前面描述的“已知项搜索”完全不同的配置。

具体来说，让我们假设我们最终遵循“以术语为中心”的方法来满足第二个需求。在这种情况下，该方法需要为“在空白上分割”参数设置一个不同的值，该值必须设置为 true。

“ **sow** 参数可以在 *SearchHandler* 级别设置，因此在查询时应用。可以在 *solrconfig.xml* 中声明它，并且根据配置，可以使用一个命名的(HTTP)查询参数覆盖它。

“空白分割”预标记化将我们带到了一个与“已知项目搜索”完全不同的场景，相反，我们“应该”进行以字段为中心的搜索；“应该”是用双引号引起来的，因为从一方面来说，如果我们实际上使用的是以字段为中心的搜索，那么从另一方面来说，我们就处于一种边缘情况，即我们用一个查询术语查询一个字段(本文中的第一个分析器**总是**输出一个术语)。

## 实施

###### 在哪里？

尽管人们可能认为首先要考虑的是如何组合这两种不同的查询策略，但在此之前，我们需要回答的问题是**在哪里实现解决方案**？显然，无论我们决定采用哪种方式，我们都必须实施一个(搜索)工作流，该工作流可以概括为下图:

![Known Item Search in Apache Solr](img/e444ad7a9ef53d1dc3565fd887ab6f50.png)

在 Solr 方面，每个“搜索”任务需要在不同的 *SearchHandler* 中执行，所以回到我们的问题:我们想在哪里实现这样的工作流？我们有三种选择:外部、中间或内部 Solr。

###### #1:客户端实施

第一种选择是在客户端应用程序中实现上面描述的流程。这假设您在这方面拥有所需的控制和编程技能。如果这个假设成立，那么编写工作流就相对容易了:您可以选择一个适用于您的语言的客户端 API 绑定，然后实现上面说明的 double +条件搜索。

*   *   **优点**:容易实现。它需要最低限度的 Solr(功能)知识。
    *   **缺点**:搜索工作流程/逻辑移到了客户端。编程是必需的，因此您必须处于这样一种环境中，在这种环境中可以完成编程，并且客户端应用程序代码在您的控制之下。

###### #2:中间人

将东西移出客户端领域，这是另一个流行的选择，仍然可以被视为客户端的替代方案(从 Solr 的角度来看)，是一个代理/适配器/外观。无论你想给这个东西取什么名字，它都是一个新的模块，位于客户端应用程序和 Solr 之间；它将拦截所有请求，并通过编排 Solr 中公开的搜索端点来实现定制逻辑。

作为一个新模块，它有几个优点:

*   *   它可以用你喜欢的语言来编码
    *   它与客户端应用程序以及 Solr 完全分离

但出于同样的原因，它也有一些缺点:

*   *   它必须被创建:设计、实现、测试、安装和维护
    *   它是您系统中的一个新部分，这必然会增加架构的整体复杂性
    *   Solr 公开了许多(索引和搜索)服务。使用这个选项，所有这些服务都应该被代理，因此导致了许多不必要的委托(即，委托服务不会给执行链增加任何价值)。

###### #3:在 Solr

最后一个选项将工作流实现(和搜索逻辑)移动到了我认为应该是 Solr 的地方。

请注意，这种选择通常不仅仅是一种“哲学”上的选择:如果你是一名搜索工程师，你很可能会被雇佣来设计、实现和调整“搜索部分”。这意味着，出于许多原因，您完全有可能将客户端应用程序视为一个外部(子)系统，在这里您没有任何控制。

这种方法的主要缺点是，正如您可以想象的那样，它需要编程技能和 Solr 内部知识。

在 Solr 中，搜索请求由一个 *SearchHandler* 使用，这个组件负责执行与给定搜索端点相关的逻辑。在我们的示例中，我们有以下符合这两个要求的搜索处理程序:

```
<!-- Known Item search -->
<requestHandler name="/known_item_search" class="solr.SearchHandler">
   <lst name="invariants">
        <str name="defType">lucene</str>
        <bool name="sow">false</bool> <!-- No whitespace split -->
        <str name="df">product_id</str>
   </lst>
</requestHandler>

<!-- Full-text search -->
<requestHandler name="/full-text-search" class="solr.SearchHandler">
    <lst name="invariants">
         <bool name="sow">true</bool> <!--Whitespace split -->
         <str name="defType">edismax</str>
         <str name="df">product_name</str>
         <str name="qf">
            product^0.7
            brand^1.5
```

除此之外，我们还需要第三个组件，负责协调上面的两个搜索处理程序。我将这个组件称为*“复合请求处理器”。*

复合处理程序还将提供由客户端调用的公共搜索端点。一旦接收到请求，复合请求处理程序就实现搜索工作流:它调用组成其链的所有处理程序，当一个调用目标产生预期的结果时，它将停止。

复合处理程序配置如下所示:

```
<requestHandler name="/search" class=".....">
    <str name="chain">/know_item_search,/full_text_search</str>
</requestHandler>
```

在客户端，只需要一个请求，因为整个工作流将通过复合请求处理程序在 Solr 中实现。换句话说，想象一个带有搜索栏的 GUI，当搜索按钮被按下时，客户端应用程序将必须检索用户输入的术语，并且只发送一个请求(到复合处理程序端点)，而不管用户的意图如何(即，不管用户属于哪个组)。

本节介绍的复合请求处理器已经实现了，你可以在我们的 Github 账号中找到，这里是[【2】](https://web.archive.org/web/20220925171301/https://github.com/SeaseLtd/invisible-queries-request-handler)。

享受吧，像往常一样，任何反馈都是热烈欢迎的！

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220925171301/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220925171301/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220925171301/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr:编排已知项目和全文搜索的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**