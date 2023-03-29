# 伦敦信息检索会议[2022 年 3 月]

> 原文：<https://web.archive.org/web/sease.io/2022/03/london-information-retrieval-meetup-march-2022.html>

![](img/366de914b828ff52c797b0bdd7366370.png)

我们非常高兴地宣布第十二届伦敦信息检索聚会，这是一次免费的**晚间聚会，旨在为对探索和讨论该领域最新趋势感兴趣的**信息检索爱好者和专业人士**提供服务。**

这一次 meetup 将是一次混合活动，既有现场活动，也有在线活动！
本次活动将由两个技术讲座**和一个问答环节&以及讲座后的一个交流环节组成。**

## **出席 MEETUP**

**地点:**伦敦 sw8 2fs 九榆树角格拉德温大厦

**需要注册**

**日期**:2022 年 3 月 30 日|下午 6:00-8:00(GMT)

## **在线聚会**

**需要注册**

**日期**:2022 年 3 月 30 日|下午 6:15-8:00(GMT)

// LONDON INFORMATION RETRIEVAL MEETUP

## 程序

// first talk

## 构建一个开源的在线学习排名引擎

相关性是主观的。“牛仔裤”查询的搜索结果中的相同项目对你我来说可能具有完全相反的价值，因为我们在尺寸、形状和口味上不同。
但是，当你想在你的排名中使用复杂的 ML 特征时，利用过去的访问者行为进行 LTR 任务通常会成为一个不那么容易的数据工程挑战。实施疯狂的事情，如滑动窗口计数器，每件商品的转换率+点击率，客户档案跟踪，在线和离线工作-你需要整个团队的 DS/DE/MLE 人员和大量的时间来粘合东西！
我们厌倦了一次又一次地重新发明 LTR 的轮子，现在向您介绍 Metarank，一种处理最典型和最复杂的数据+特征工程任务的开源个性化服务。它监听描述访问者行为的事件流，将其映射到最常见的 ML 特性，并实时重新排序项目，以像 CTR 一样最大化目标。您所需要的只是一个 YAML 配置和一点 JSON I/O

// our speakers![](img/893dafdc3406dee6a659a39ed3b1630b.png)[](https://web.archive.org/web/20220929233143/https://www.twitter.com/public_void_grv)*[](https://web.archive.org/web/20220929233143/https://www.linkedin.com/in/romangrebennikov/)* **#### [罗曼·格雷本尼科夫](#)**  **Findify 的 ML 工程师，致力于搜索个性化和推荐。函数式编程、学习排序模型和性能工程的实用爱好者。** **![](img/81c06efc8ca88ebfc7825273d49a1425.png)[](https://web.archive.org/web/20220929233143/https://www.linkedin.com/in/vgoloviznin/) *#### [Vsevolod Goloviznin](#)*  *过去的软件工程师，转行是为了更贴近客户和产品。拥有多年与客户沟通的经验，了解他们真正想要什么，并作为产品负责人将这些信息传达给工程师。*** ***// video

[https://web.archive.org/web/20220929233143if_/https://www.youtube.com/embed/KDtNHjgxS6s](https://web.archive.org/web/20220929233143if_/https://www.youtube.com/embed/KDtNHjgxS6s)

// second talk

## 如何缓存您的搜索:一个开源实现

IT 系统中使用缓存将数据存储在专用结构中，以便快速访问，从而可以更快地满足未来的请求。它们是信息检索系统中存储查询结果和加速未来查询执行的有效工具。像 Apache Solr 这样的开源系统使用三种不同的缓存:queryResultCache、filterCache 和 documentCache。
在本次演讲中，我们将重点介绍 queryResultCache 和 filterCache，并通过实际例子了解它们如何用于处理不同类型的查询。

// our speaker![](img/42c03d97b2e33b3b3ff6ea6cdc3ac464.png)[](https://web.archive.org/web/20220929233143/https://mobile.twitter.com/dantuzi)*[](https://web.archive.org/web/20220929233143/https://www.linkedin.com/in/daniele-antuzi-6773a282/)* **#### [丹尼尔·安图兹](https://web.archive.org/web/20220929233143/https://sease.io/about/daniele-antuzi)

SOFTWARE ENGINEER @ Sease** **// slides

[https://web.archive.org/web/20220929233143if_/https://www.slideshare.net/slideshow/embed_code/key/gUE7c4Omdmnd4c](https://web.archive.org/web/20220929233143if_/https://www.slideshare.net/slideshow/embed_code/key/gUE7c4Omdmnd4c)

**[How to cache your searches_ an open source implementation.pptx](https://web.archive.org/web/20220929233143/https://www.slideshare.net/SeaseLtd/how-to-cache-your-searches-an-open-source-implementationpptx "How to cache your searches_ an open source implementation.pptx")** from **[Sease](https://web.archive.org/web/20220929233143/https://www.slideshare.net/SeaseLtd)**// video

[https://web.archive.org/web/20220929233143if_/https://www.youtube.com/embed/WJydOmPlScs](https://web.archive.org/web/20220929233143if_/https://www.youtube.com/embed/WJydOmPlScs)*****