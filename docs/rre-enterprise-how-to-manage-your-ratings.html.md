# RRE 企业报:如何管理你的评级

> 原文：<https://web.archive.org/web/sease.io/2022/06/rre-enterprise-how-to-manage-your-ratings.html>

任何搜索质量评估过程的第一步都是定义基本事实:
一组三元组<查询、文档、评级>指定文档与特定查询的相关程度。
评级可以通过**显式**或**隐式**方式产生:

*   *   **显式**–一组人类评委(希望是领域专家)为<查询、文档>对分配评级
    *   **隐式**–收集用户交互(点击、下载、添加到购物车、添加到收藏夹……)来估计<查询、文档>对的相关性评级

RRE 企业生态系统支持这两者。
让我们看看如何在本教程中:

![](img/06af6d36410b09862ab4b63a18850e85.png)

*Ratings Tab*

要开始配置您的评分，您需要点击**设置按钮**并选择**“评分”选项卡**。
这为您打开了三个功能:

T35

1.  1.  **上传**一个 JSON 文件，其中包含 RRE 开源格式的评级
    2.  **从隐式交互中生成**评级
    3.  **查看**评分并编辑它们

## 上传

**视频:Ratings1** 
如果您过去使用过 Rated Ranking 评价，那么您应该熟悉 rating 文件支持的 JSON 格式:

```
{
  "tag": "<string>",
  "index": "<string>",
  "corpora_field": "<string>",
  "id_field": "<string>",
  "topics": [
    {
      "description": "<string>",
      "query_groups": [
        {
          "name": "<string>",
          "queries": [
            {
              "template": "<string>",
              "placeholders": {
                "$key": "<value>",
              }
            }
          ],
          "relevant_documents": [
            {
              "document_id": {
                "gain": "<number>"
              }
            }
          ]
        }
      ]
    }
  ],
  "query_groups": [
    {
      "name": "<string>",
      "queries": [
        {
          "template": "<string>",
          "placeholders": {
            "$key": "<value>",
          }
        }
      ],
      "relevant_documents": [
        {
          "document_id": {
            "gain": "<number>"
          }
        }
      ]
    }
  ],
  "queries": [
    {
      "template": "<string>",
      "placeholders": {
        "$key": "<value>",
      }
    }
  ]
}
```

*   *   `**tag**`:与此等级集相关联的唯一标识符(您可以稍后通过标签检索一组等级)
    *   `**index**`:弹性搜索**中的索引名**/Solr**中的集合名**
    *   `**corpora_file**`:包含评级所基于的信息语料库的文件(要索引的 **Solr** 或 **Elasticsearch** 文档)。如果您想要启动嵌入式评估目标搜索引擎，这是必要的。
        如果您的评估目标是现有的**elastic search**/**Solr**(其索引已经存在)您不需要这个
    *   `**id_field**`:在**elastic search**mapping/**Solr**模式中的唯一标识符字段
    *   `**topics**` : *可选的*主题和/或查询组列表
        *   `description`:对内部查询组进行分组的主题标题
        *   `query_groups`:以相同名称和主题分组的查询列表
            *   `name`:查询组标识
            *   `queries`:该组要执行的查询列表
                *   `template`:该查询使用的模板的字符串名称
                *   占位符:模板中要替换的键值对的对象
        *   `relevant_documents`:列出带有映射文档的对象，以获得或关联值
    *   `**query_groups**` : *可选*有相关查询的对象列表。
    *   `**queries**`:必需*用于评估的模板和占位符替换的对象列表。
        *   *如果`topics`和`query_groups`没有定义
            和

**视频:Ratings2 我们的审判收集者浏览器插件为您提供保护！关于这个组件的博客即将发布。**

## 生成(通过隐式交互)

到目前为止，我们已经看到了如何管理显式评级，但是隐式评级呢？
RRE 企业版为您提供了收集用户互动并根据可配置的指标生成评分的能力。

一个<查询，文档>交互用 JSON 表示，可以由 RRE 的一个专用端点收集——企业:

```
POST http://localhost:8080/1.0/rre-enterprise-api/input-api/interaction
```

```
{
	"collection": "papers",
	"query": "interleaving",
	"blackBoxQueryRequest": "GET /query?q=interleaving HTTP/1.1\r\nHost: localhost:5063",
	"documentId": "1",
	"click": 1,
	"timestamp": "2021-06-23T16:10:49Z",
	"queryDocumentPosition": 0
}
```

*   `**collection**`:弹性搜索**中的索引名**/Solr**中的集合名**
*   `**query**`:人类可读的查询表示。大多数情况下是自由文本查询
*   `**blackBoxQueryRequest**`:与查询相关联的完整 http 请求。大多数时候它是对 search-API 层的请求
*   `**documentId**`:文件的唯一标识符(必须与**elastic search**/**Solr**中使用的标识符相同)
*   `**impression**`:1–如果这种交互是一种印象(响应查询向用户显示的文档)
*   `**click**`:1–如果该交互是响应查询的文档点击
*   `**addToCart**`:1–如果此交互是响应查询而将文档添加到购物车
*   `**sale**`:1–如果此交互是响应查询的文档销售
*   `**revenue**`:<int>–与响应查询的文档销售相关的收入金额
*   `**timestamp**`:交互发生的时间
*   `**queryDocumentPosition**`:响应查询的文档在搜索结果列表中的位置

**视频:收视率 4 -5** (菲诺 3:17)

一旦用交互填充了 RRE-企业，就可以使用生成功能来估计<query document="" rating="">三元组:</query>



![](img/c65f3625500cc98589b960ba26da69c1.png)

*   **在线指标**指定使用什么来估计相关性评级
    ，例如
    选择点击率意味着 RRE 企业将使用某个<查询、文档>对的印象和点击来估计相关性评级，将该值与收集的交互中所有查询的最小值和最大值进行比较
*   **过滤查询**允许通过任何属性(集合，日期..)
*   **标签**是与我们正在生成的评级集相关联的唯一标识符

T13

**视频:收视率5**fino a 6:04–收视率 6

## 视角

上传或生成评分后，您可以查看和编辑它们。
首先，您可以检索由标签设置的等级:

![](img/88ea974394288bdbe1e5b84172d041a6.png)

每一排都是单个<query document="" rating="">三联体。
点击评分，您可以编辑评分值并自动保存。
点击查询显示三联体的详细信息。</query>

您还可以按收藏、主题和查询组过滤您的评级，以优化您的导航。

![](img/3f7ee7e732171cbc0cd07f13937359bd.png)

**视频:收视率 3**

[https://web.archive.org/web/20221202230945if_/https://www.youtube.com/embed/X_5xIcOKPLg](https://web.archive.org/web/20221202230945if_/https://www.youtube.com/embed/X_5xIcOKPLg)

// begin your journey into the search quality evaluation

## 评级评估企业

[DISCOVER MORE AND DOWNLOAD](https://web.archive.org/web/20221202230945/https://sease.io/rated-ranking-evaluator-enterprise)// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 RRE 企业以及如何管理你的评分的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！