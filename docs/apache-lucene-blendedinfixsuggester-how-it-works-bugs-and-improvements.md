# Apache Lucene BlendedInfixSuggester:工作原理、缺陷和改进

> 原文：<https://web.archive.org/web/sease.io/2018/06/apache-lucene-blendedinfixsuggester-how-it-works-bugs-and-improvements.html>

Apache Lucene/Solr 建议对 Sease 很重要:我们在过去探索过这个主题，我们坚信自动完成功能对许多搜索应用程序来说是至关重要的。
这篇博文详细探讨了 Lucene BlendedInfixSuggester 的当前状态、最新版本的一些错误(附带解决方案)以及一些可能的改进。

## BlendedInfixSuggester

BlendedInfixSuggester 是 AnalyzingInfixSuggester 的一个扩展，增加了在匹配的文档中对查询的前缀匹配进行加权的功能。
如果点击越接近建议的开始，得分越高。
**注意:**在当前阶段，只有查询中的第一项会影响建议得分

Let’s see some of the configuration parameters from the official wiki:

*   *   **blenderType** :使用第一个匹配字的**位置计算位置权重系数。可以是下列值之一:

        *   **position _ linear**:weightFieldValue *(1–0.10 * position):匹配到起点将被给予更高的分数(默认值)
        *   **position _ reciprocal**:weightFieldValue/(1+position):匹配到起点将被给予比线性衰减更快的分数
        *   **position _ exponential _ reciprocal**:weightFieldValue/pow(1+position，指数):匹配到起点默认 2.0。** 

| 描述 |
| **数据结构** | 辅助 Lucene 指数 |
| **建筑** | 对于每个文档，根据*suggestAnalyzerFieldType*对**存储的内容**进行**分析**，然后再对 *EdgeNgram* 令牌进行过滤*。*最后，使用这些标记构建一个辅助索引。 |
| **查找策略** | 根据 *suggestAnalyzerFieldType* 分析查询。然后，针对**辅助 Lucene 索引**触发短语搜索，从字段内容中每个标记的**开头开始识别建议。** |
| **建议返回** | 字段的**全部内容**。 |

这种建议器现在非常普遍，因为它允许在字段内容中间提供建议，利用了字段提供的分析链。通过这种方式，可以提供考虑到**同义词**、**停用词、词干**和分析中使用的任何其他标记过滤器的建议，并基于**内部标记**匹配建议。最后，根据职位匹配情况对建议进行评分。示例的简单文档集如下:

```
[
      {
        "id":"44",
        "title":"Video gaming: the history"},
      {
        "id":"11",
        "title":"Nowadays Video games are a phenomenal economic business"},
      {
        "id":"55",
        "title":"The new generation of PC and Console Video games"},
      {
        "id":"33",
        "title":"Video games: multiplayer gaming"}]
```

以及一个简单的同义词映射:多人在线

让我们看一些例子:

| 自动完成的查询 | 建议 | 说明 |
| *《博彩》* | 

*   【视频 **gami** ng:历史】
*   “视频**游戏**:多人游戏”
*   “如今视频游戏是一项惊人的经济业务”…

 | 对输入查询进行分析，产生的令牌如下:**“game”。**在辅助索引中，对于每个字段内容，我们有 EdgeNgram 标记:“v”、“vi”、“vid”…、“g”、“ga”、“gam”、“T28”、“game”。所以匹配发生了，建议被返回。**注意:**前两个建议的排名较高，因为匹配的术语碰巧更接近建议的开头 |

让我们看看给定不同搅拌机类型的每个建议的得分:

| **查询** | *游戏* |
| 建议 | 首次位置匹配 | 位置线性 | 位置倒数 | 位置指数倒数 |
| gami ng:历史 | *1* | 1-0.1*position = **0.9** | *1/(1+位置)= 1/2 = **0.5*** | *1/(1+position)^2 = 1/4 =**0.25**t49】* |
| 视频**游戏**:多人游戏 | *1* | 1-0.1*position = **0.9** | *1/(1+位置)= 1/2 =**0.5*** | *1/(1+位置)= 1/4 = **0.25*** |
| 如今，视频游戏是一项惊人的经济业务 | *2* | 1-0.1*position = **0.8** | *1/(1+位置)= 1/3 = **0.3*** | *1/(1+位置)= 1/9 =**0.1*** |

建议的最终得分为:

```
long score = (long) (weight * coefficient)
```

**注意**我强调数据类型的原因是它直接影响我们讨论的第一个 bug。

## 建议分数近似值

可选的 weightField 参数对于混合中缀建议器非常重要。
分配建议权重的值(从上述字段中提取)。
**例如** 建议可能来自产品名称字段，但建议权重取决于所建议产品的利润。

<str name="”field”">产品名称</str>
<str name = " weight field ">利润< /str >

到目前为止，一切顺利，但不幸的是，这有两个问题。

## 错误 1 -未定义 WeightField>零建议分数

**如何重现**:不要在建议者配置
中定义任何 weight field**效果**:建议排名丢失，所有建议都是 0 分，匹配的位置不再重要
weight field 不是 BlendedInfixSuggester 的强制配置。
您的用例不能包含您的建议的任何权重，您只对位置评分感兴趣(BlendedInfixSuggester 首先存在的主要原因)。
不幸的是，目前这是不可能的:
如果没有定义 weight 字段，每个建议的**权重将为 0** 。
这是因为与文档字典中的每个文档相关联的权重很长。如果要从中提取权重的字段未定义(null)，则返回的权重将仅为 0。
这不允许区分权重何时应该为 0(从字段中提取的值)或空(根本没有值)。
这里提出了一个解决方案[【3】](https://web.archive.org/web/20220929233558/https://issues.apache.org/jira/browse/LUCENE-8343)。

## Bug 2 -小权重的建议分数近似值不佳

在建议的分数计算中存在误导性的数据类型转换:

```
long score = (long) (weight * coefficient)
```

如果与建议相关联的权重是单一的或足够小，这种看似无害的转换实际上会带来非常恶劣的影响。

**权重** =1
视频 **gami** ng:历史
1-0.1 *位置=**0.9 * 1 = cast = 0** *1/(1+位置)= 1/2 =**0.5 * 1 = cast = 0*** *1/(1+position)^2 = 1/4 =**0.25 * 1 = cast = 0***

**权重** =2
视频 **gami** ng:历史
1-0.1 *位置=**0.9 * 2 = cast = 1** *1/(1+位置)= 1/2 =**0.5 * 2 = cast = 1*** *1/(1+position)^2 = 1/4 =**0.25 * 2 = cast = 0***

基本上你冒着失去你的建议排名的风险，把分数降低到只有几个可能的值:0 或 1(在边缘情况下)

这里提出了一个解决方案[【3】](https://web.archive.org/web/20220929233558/https://issues.apache.org/jira/browse/LUCENE-8343)

## 多项匹配处理

在自动完成查询中有多个术语是很常见的，因此您的建议者应该能够相应地管理建议中的多个匹配。

给定一个简单的语料库(仅由以下建议组成)和查询:
**【Mini Bar Frid】**

你会看到这些建议:

*   *   **1000 |迷你吧**什么的 **Frid** ge
    *   **1000 |迷你吧**别的东西 **Frid** ge
    *   **1000 |迷你吧冰箱**什么的
    *   **1000 |迷你吧冰箱**别的
    *   **1000 |迷你**有事**吧友**葛

这是因为此时，**第一个匹配项赢得所有**(其他位置被忽略)。这带来了许多可能的联系(1000)，应该打破给用户一个好的和直观的排名。

但是凭直觉，我希望得到类似这样的结果(注意 allTermsRequired=true，模式权重字段总是返回 1000)

*   *   **迷你吧 Frid** 葛什么的
    *   迷你酒吧的朋友们，来点别的吧
    *   **迷你酒吧**有事 **Frid** 葛
    *   **迷你酒吧**别的东西 **Frid** ge
    *   **迷你**有事**吧 Frid** 葛

让我们来看一个提议的解决方案[【4】](https://web.archive.org/web/20220929233558/https://issues.apache.org/jira/browse/LUCENE-8347):

## 位置系数

除了只考虑建议中的第一个术语位置，还可以使用匹配术语[“mini”、“bar”、“冰箱”]中的所有匹配位置。
每个位置的匹配都会影响得分:

*   *   匹配的术语位置离理想的位置匹配有多远
        *   **查询**:迷你吧 Fri，**理想位置**:【0，1， **2**
        *   **建议 1** : **迷你吧**有事 **Fri** dge，**匹配位置:**【0，1， **3** ]
        *   **建议 2** : **迷你吧**别的 **Fri** dge、**匹配位置:**【0，1， **4** ]
        *   建议 2 将被扣分为“Fri”比赛发生时距离理想位置4>3**2**
    *   错误位置发生得越早，罚分越重
        *   **查询**:迷你吧 Fri，**理想位置** : [0，1，2]
        *   **建议 1** : **迷你吧**有事 **Fri** dge、**匹配位置:**【0，1， **3** ]
        *   **建议二** : **迷你**有事**吧**Fridge、**匹配职位:**【0、 **2** 、 **3**
        *   建议 2 将被额外扣分，因为位置 **栏** 中的第一个不匹配更接近建议 的开始

仅考虑**折扣位置**证明有用:

**Query1** :酒吧点点
**Query2** :点点
**建议**:迷你酒吧点点冰箱
**查询 1 建议匹配项位置**:【1，2】
**查询 2 建议匹配项位置**:【2】

如果我们比较这两个查询的建议分数，仅仅因为第一个查询匹配 2 个词(连续的)而第二个查询只有一个匹配(比查询 1 中的第一个匹配位置更差)就惩罚第一个查询似乎是不公平的

引入这种先进的位置系数演算有助于改进所创建的实验测试的整体行为。
获得的结果很有希望:

**查询**:小酒吧 Fri
100 | **小酒吧 Fri**dge something
100 |**小酒吧 Fri**dge something
100 |**小酒吧 Fri**dge a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a a
26 |**小酒吧** something **Fri** dge **迷你酒吧**其他东西 **Fri** dge
17 | **迷你**其他东西**酒吧 Fri**dge
8 |其他东西**迷你酒吧冰箱**
7 |其他东西**迷你酒吧冰箱**

精确的前缀匹配仍然有差距，但是让我们看看我们是否也能完成改进。

## 令牌计数系数

让我们来关注一下我们刚刚看到的前三个排名建议:

**查询**:迷你吧 Fri
**100** | **迷你吧 Fri**dge something
**100**|**迷你吧 Fri**dge something
**100**|**迷你吧 Fri**dge a a a a a a a a a a a a a a a a a a a a a a a a a a

直觉上，我们希望这个订单能打破这种束缚。
匹配项的数量与建议项的总数越接近越好。
理想情况下，我们希望我们的最高评分建议尽可能只包含匹配的术语。我们也不想给其他建议带来强烈的不一致，我们应该只影响纽带。
这可以通过计算附加系数来实现，取决于术语计数:
**令牌计数系数** =匹配术语计数/总术语计数

然后我们可以相应地调整这个值:
最终分数的 90%将来自位置系数
最终分数的 10%将来自令牌计数系数

**查询**:迷你吧 Fri
90 * 1.0+10 * 3/4 =**97**|**迷你吧 Fri**dge something
90 * 1.0+10 * 3/5 =**96**|**迷你吧 Fri**dge something
90 * 1.0+10 * 3/25 =【t8

这将需要一些额外的调整，但总的想法应该是当涉及多个术语匹配时，给 BlendedInfix 带来更好的排名功能！
如果你有任何**建议，**欢迎在下面留言！
代码可以在 Lucene 吉拉发布[【4】](https://web.archive.org/web/20220929233558/https://issues.apache.org/jira/browse/LUCENE-8347)附带的 Github Pull 请求中找到。

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929233558/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929233558/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929233558/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Lucene BlendedInfixSuggester 的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！