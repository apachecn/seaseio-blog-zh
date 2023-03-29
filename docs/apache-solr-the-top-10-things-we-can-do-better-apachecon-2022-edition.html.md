# Apache Solr:我们可以做得更好的 10 件事——Apache con 2022 版

> 原文：<https://web.archive.org/web/sease.io/2022/10/apache-solr-the-top-10-things-we-can-do-better-apachecon-2022-edition.html>

ApacheCon North America 2022 于 10 月 6 日星期四在新奥尔良(路易斯安那州)落下帷幕，这是一次精彩的会议，充满活力，发人深省。
特别是对于 Apache Solr 社区来说，这是一个聚会的美妙时刻。
我个人很高兴在会议期间和会议结束后有许多 Lucene/Solr 成员的陪伴，这使得这些活动更加特别。
由 **David Smiley** 组织的 Apache Solr Birds of Feather(BoF)是探索用户和贡献者——想要成为我们心爱项目的人——的基础，哦，天哪，我们发现了多少！18 个人参加了会议，7 个人只对 Apache Lucene 感兴趣，11 个人有一些(或很多)Apache Solr 的经验。
作为 Apache Lucene/Solr 提交者和 PMC 成员，我的首要任务是提高项目的质量，并让社区更多地参与进来，所以让我们来探讨一下我们最终确定的当前 Apache Solr 的 10 大难点！

// 10

## 管理本体

有一段时间，另一个 Apache 项目正在兴起，它是文本丰富的本体管理之王，它的名字是 [Apache Stanbol](https://web.archive.org/web/20221130070345/https://stanbol.apache.org/) 。
我在这里用的是过去，因为这个项目已经退役，所以不再维护了。
当这种情况发生时，很难通过开源软件来填补空白，Apache Solr 的插件/扩展可能会被发现。
一个专用的 Apache Solr 集合/托管资源来存储/搜索知识库中的本体(可能直接导入三元组存储/RDF)怎么样？)，可能提供与更新请求处理器结合使用的标记服务(可能是[文本标记器](https://web.archive.org/web/20221130070345/https://solr.apache.org/guide/solr/latest/query-guide/tagger-handler.html) 2.0)？
这个领域最近没有被积极研究和开发，但这意味着有机会通过新的贡献大放异彩！

// 9

## (Performant？)加入

严格来说，Apache Lucene 和 Solr 都实现了连接，并提供了运行嵌套文档搜索([查询](https://web.archive.org/web/20221130070345/https://solr.apache.org/guide/solr/latest/query-guide/join-query-parser.html) / [索引](https://web.archive.org/web/20221130070345/https://solr.apache.org/guide/solr/latest/query-guide/block-join-query-parser.html)时间)的能力，包括跨索引和不同实例的集合。
这绝对是一个令人惊叹的特性，但考虑到性能影响，它们一直被推荐为最后的手段。
很多用户都在使用(并且可能滥用？)这个功能反正肯定有困难。
可能是时候在这个方向上大力投资，并改进/重新设计该功能的一部分了。

// 8

## 核心功能展示

当然，有 [Apache Solr 参考](https://web.archive.org/web/20221130070345/https://solr.apache.org/guide/solr/latest/)指南，这是一个巨大的文档来源，包括许多教程和如何使用某些功能的示例，但有没有一个核心部分包含最常见的用例以及清晰、详细的示例，比如类固醇上的[“入门”](https://web.archive.org/web/20221130070345/https://solr.apache.org/guide/solr/latest/getting-started/introduction.html)，可能有额外的代码示例和由社区管理的实时 Solr 实例以进行交互？

// 7

## 集合/节点级别的指标

目前，可以详细调查在核心级别发生了什么，但在集合级别(分布式)或节点级别(汇总所有核心)收集信息和统计数据并不容易。
这应该特别关注使用统计数据，例如请求处理程序总数收到的查询请求数量和索引吞吐量。

// 6

## 寻找新的工作议题

我们正在触及主要的痛点:Apache 项目首先是社区，然后是技术解决方案，所以我们应该帮助新人进入项目。
除了开始编码，进入软件项目还有什么更好的方法？(好吧，可能这个观点有失偏颇，我毕竟是个软件工程师:))

目前，在 Apache Solr 吉拉中有一个[“new dev”](https://web.archive.org/web/20221130070345/https://s.apache.org/newdevsolr)标签，但是有多少人知道呢？(比如我没有)。
是的，这里有解释([https://CWI ki . Apache . org/confluence/display/Solr/howtocontribute](https://web.archive.org/web/20221130070345/https://cwiki.apache.org/confluence/display/solr/howtocontribute))，但是我在 [Apache Solr reference](https://web.archive.org/web/20221130070345/https://solr.apache.org/guide/solr/latest/) 指南上找不到任何参考(请允许我这个词双关)。
我们可能需要更多，也许是不同层次的问题(新开发层 1，新开发层 2…)？随着难度的增加？因此，根据新贡献者希望/能够贡献的时间量，可以选择。
我们肯定应该更新参考指南，并在会议和集会上宣传这一点。

// 5

## 可解释性

为什么**单据 A** 被我查询返回？为什么**文件 B** 在第 7 位而**文件 A** 在第 3 位？
我们所有从事搜索的人都多次听到过这些问题和观察结果。但是，对于经验丰富的工程师来说，解释为什么一份文件被退回并带有一定的分数也是相当困难的。
当然，还有[调试结果](https://web.archive.org/web/20221130070345/https://solr.apache.org/guide/solr/latest/query-guide/common-query-parameters.html#debug-parameter)功能，这很棒，但是不容易阅读。
[一(可能退役？)Chrome 插件](https://web.archive.org/web/20221130070345/https://www.linkedin.com/pulse/debugging-solr-queries-using-chrome-extension-leonardo-foderaro/)是一个很棒的解决方案(我们应该把它集成到 Apache Solr admin UI 代码库中！)和[http://spliner . io](https://web.archive.org/web/20221130070345/http://splainer.io/)是一个外部服务，旨在帮助解释，但我们可以做得更好……
用户(或超级用户)应该能够直接从 Apache Solr 本身获得所需的一切，这里肯定有贡献的空间(和玩新颖策略的许多乐趣)。

// 4

## 向后兼容性

在观众中，有两个人表达了对 Apache Solr 版本的向后兼容性的担忧，这可能会阻碍用户和企业升级(并通过采用和潜在的 bugs 改进发现来减缓项目的进度)。我们一定要让升级变得尽可能的简单，谨慎的向后兼容是至关重要的。我个人认为我们已经做得很好了，但是如果人们注意到了，还有改进的空间！

// 3

## 做事情的方式太多了

这里也投两票，在 Apache Solr 中，开发中的增量迭代带来了解决相同问题的许多不同方式。
刻面？高亮？分组(或折叠)？更多…
大多数时候，每个功能的每个新迭代都带来了改进和新功能，但却无法实现最初实现做得如此之好的“小事情…
这样，我们最终积累了巨大的灵活性，但当新用户到来并需要在多个实现中选择他/她需要的功能时，也会产生巨大的困惑。
现在是不是应该放弃更多，或者花费额外的精力来合并一些不同的实现，只获得那些世界中最好的？

// 2

## Lucene/Solr 文档

三票缓解进入 Lucene/Solr 世界的陡峭学习曲线。
参考指南很棒，每个版本都会更新，但是没有深入到内部。
书籍对理解核心和内在有很大帮助，但很快就会过时。代码库是巨大的，有一些好的 Javadocs，但是很难导航。
这里有哪些选项？
可能是托管的书，由社区详细描述和维护。我们是否需要更多的例子和代码，也许直接放在相关的 java 包或演示文件夹中？
测试是好的，但绝对不是深入了解和理解一个特性的正确方法。

我们应该区分用户路径和开发者路径吗？
毫无疑问，这里有很多需要改进的地方，为新的贡献留有很多余地(这将使新的贡献变得容易，这是一个积极的自我强化循环！)

// 1

## 贡献和获得关注

最后，我们到达了 Apache Solr 当前面临的最大困难:投稿并获得足够的关注以获得评论并合并投稿。来自观众中的四个人提出了这个问题:即使你最终为一个新的功能/存在的错误提供了一个工作补丁，如果你已经不在社区中或者不认识合适的人，你真的很难得到必要的关注。
的确，所有的提交者和 PMC 成员都是志愿者，但很遗憾一些新的贡献者有这种感觉，我们应该更热情、更细心。
当然，我们鼓励新的贡献者做他们的功课:简短的贡献、激光聚焦的细节、测试和 GitHub pull requests 标记为审查者，那些最近接触过代码领域的人肯定会有所帮助。
从机构群体的角度来看，我们可以致力于更频繁的计划和会议，提交者可以参与其中，贡献者可以明确他们感兴趣的问题。
特别是在像 ApacheCon 这样的会议期间，增加一个黑客马拉松会前/会后会议将有助于解决这些问题。
[Search Solutions’22](https://web.archive.org/web/20221130070345/https://www.bcs.org/membership-and-registrations/member-communities/information-retrieval-specialist-group/conferences-and-events/search-solutions/search-solutions-2022/)将于 11 月 22 日和 23 日在伦敦举行，届时[伦敦信息检索会议将在附近举行](https://web.archive.org/web/20221130070345/https://www.meetup.com/it-IT/London-Information-Retrieval-Meetup-Group/)(我们正在与我们的合作伙伴敲定日期)，这可能是英国贡献者的一个好机会。

###### 结论

你怎么想呢?2022 年 Apache Solr 的主要痛点是什么，我们可以做些什么来改善？
在评论中告诉我们吧！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr:我们可以做得更好的 10 件事 ApacheCon 2022 版的帖子吗？不要忘记订阅我们的时事通讯，以便在信息检索世界中保持最新状态！