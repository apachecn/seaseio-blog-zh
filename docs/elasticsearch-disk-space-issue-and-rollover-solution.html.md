# 弹性搜索磁盘空间问题和翻转解决方案

> 原文：<https://web.archive.org/web/sease.io/2022/06/elasticsearch-disk-space-issue-and-rollover-solution.html>

嗨读者们！

这篇博客想帮助那些在 Elasticsearch 中遇到索引编写器磁盘空间问题的人。

让我们从第一个观察开始，**我们所指的索引编写器磁盘空间问题是什么？**



当处理大量数据时，出现与磁盘相关的错误并不罕见。如果索引的文档太多，磁盘空间会饱和，导致 **BulkIndexError** 。这是我们将获得的日志消息:

```
[ERROR] BulkIndexError: ('500 document(s) failed to index.', [{'index': {'_index': 
'your_index_name', '_type': '_doc', '_id': 'vAAt1H8BpDNDRA5qgPEv', 'status': 403, 'error': 
{'type': 'cluster_block_exception', 'reason': 'index [your_index_name] blocked by: 
[FORBIDDEN/8/index write (api)];’}
```

这个错误告诉我们什么？
错误报告写操作被阻止，因此无法在 *your_index_name* 索引中索引新文档。当磁盘空间消耗达到总磁盘大小的 80%时，Elasticsearch 会自动设置该块。



您可以通过 GET settings API 检查当前的索引块状态:

```
GET https://localhost:9200/your_index_name/_settings
```

下面是在获得的响应中要查看的参数:

```
{
    "your_index_name": {
        "settings": {
            "index": {
                ...
                **"blocks": {
                    "write": "true"
                },**
                "number_of_shards": "2",
                "provided_name": "your_index_name",
                "creation_date": "1650529887595",
                "number_of_replicas": "1",
                "uuid": "2FPWsd-LMHyQSwXaM523GA",
                "version": {
                    "created": "7100299"
                }
            }
        }
    }
}
```

这篇博文的目的是回答这两个问题:

T45

1.  1.  ***这个问题出现时怎么解决？***
    2.  ***如何防止问题再次发生？***

###### 临时解决方案

这是释放磁盘空间和删除索引块的临时解决方案。

这里我们想描述一种方法，它将帮助您删除无用的文档，以便释放部分磁盘空间，并避免索引块设置。

首先，我们通过这个请求手动删除索引块:

```
PUT /[_all|<your_index_name>]/_settings
{
  **"index.blocks.write": null**
}
```

这是必要的，因为删除也是一个写操作。

此时，我们可以删除一些文档。**小心，从少量开始，因为该操作会暂时增加磁盘消耗**(直到段合并发生，被删除文档的空闲空间被占用)。

删除后，我们可以通过以下方式检查当前的磁盘使用情况:

```
GET https://localhost:9200/_cat/allocation?v&pretty
```

以下是获得的响应:

| `shards` | `disk.indices` | `disk.used` | `disk.avail` | `disk.total` | `disk.percent` | `host` | `ip` | `nod` e |
| `72` | `11.9gb` | `17gb` | `81.2gb` | `98.3gb` | `17` | `x.x..` | `x.x.`.`` | `645784hwe...` |
| `72` | `11.9gb` | `17gb` | `81.2gb` | `98.3gb` | `17` | `x.x.`.`` | `x.x.`.`` | `374562gfi...` |

如果我们仍然需要空闲空间，我们可以重复这个过程。
如果擦除导致磁盘消耗超过 80%,则需要再次解除锁定。

我们称之为“临时解决方案”,因为它帮助我们释放磁盘并删除索引块，但是**它不能避免错误在特性中再次出现。**
为了做到这一点，我们建议自动管理索引。

###### 索引生命周期-滚动解决方案

来自 Elasticsearch 文档:
*“指数生命周期管理(ILM)*[【1】](https://web.archive.org/web/20221202235827/https://www.elastic.co/guide/en/cloud/current/ec-configure-index-management.html)*Elastic Stack 的功能提供了一种集成和简化的方法来管理基于时间的数据，从而更容易遵循管理指数的最佳实践。与索引监管相比，迁移到 ILM 可以让您对每个索引的生命周期进行更精细的控制。”*

*“您可以配置索引生命周期管理(ILM)*[【2】](https://web.archive.org/web/20221202235827/https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)*策略，根据您的性能、弹性和保留要求自动管理索引。例如，你可以用 ILM 来:*

*   *   S *当一个索引达到一定的大小或文档数量时，钉上一个新的索引*
    *   *每天、每周或每月创建一个新的索引，并存档以前的索引*
    *   *删除过时的索引以实施数据保留标准*

*您可以通过 Kibana Management 或 ILM API 创建和管理索引生命周期策略。”*

这正是我们避免磁盘问题所需要的。当一个索引达到一定的大小时，我们希望建立一个新的索引(I)并删除旧的索引(II)。

第一个要求(I)可以通过**翻转策略**实现，即:*“当现有指标满足一个或多个翻转条件时，将目标翻转到新指标”*[【3】](https://web.archive.org/web/20221202235827/https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-rollover.html)；而第二个要求(II)可以通过策略的**定义**来完成。在策略中，我们可以定义状态、动作和转换。
与该策略相关联的索引从默认状态开始，处理该状态下的动作，并评估转换条件。如果条件为真，索引将传递到新状态，将再次执行动作和转换。

来自 Elasticsearch 文档:*“一个指数的生命周期政策规定了哪些阶段是适用的，在每个阶段执行什么动作，以及何时在阶段之间转换”*[【4】](https://web.archive.org/web/20221202235827/https://www.elastic.co/guide/en/elasticsearch/reference/current/ilm-index-lifecycle.html)。

让我们一起来看看创建一个自动化索引翻转和删除的策略的所有必要步骤。对于我们的例子，我们使用的是由 Amazon 管理的 Elasticsearch 7.10 版本，它包含了开放的发行版插件。
我们利用 **Kibana** 工具来完成其中的一些步骤。

###### 
1–创建用于滚动更新的索引模板

该模板是用于翻转的模板。这定义了新创建的索引将具有的设置。

*   *   在 *index_patterns* 中，我们定义了新索引的名称。在这种情况下，它从 *index_name-* 开始，然后是一个递增的数字。
    *   *index . open distro . index _ state _ management . rollover _ alias*是翻转别名的名称。这与当前活动索引(用于索引新文档的索引)相关联，并将在翻转完成后移动到新索引。

下面是创建**索引模板**的请求:

```
PUT /_index_template/template-name
{
  "index_patterns": ["index_name-*"],
  "template": {
   "settings": {
    "index.opendistro.index_state_management.rollover_alias": "rollover-alias-name"
   }
 }
}
```

**注意**如果你还没有打开发行版，这些是等价的索引设置:
**elastic search**:index . life cycle . rollover _ alias
**Open search**:index . plugins . index _ state _ management . rollover _ alias

###### 
2–创建策略



第二步是**策略定义**。
您可以在 Kibana 中添加此内容，方法是进入:



Kibana →索引管理→状态管理策略→创建

此时，需要一个策略 ID 和策略定义。

这里有一个例子:

```
{
    "policy_id": "**rollover_policy**",
    "description": "Rollover policy for index_name-* indexes.",
    "last_updated_time": 1650529799079,
    "schema_version": 1,
    "error_notification": null,
    "default_state": "**hot_state**",
    "states": [
        {
            "name": "**hot_state**",
            "actions": [
                {
                    "rollover": {
                        "min_doc_count": 100
                    }
                }
            ],
            "transitions": [
                {
                    "state_name": "warm_state"
                }
            ]
        },
        {
            "name": "**warm_state**",
            "actions": [],
            "transitions": [
                {
                    "state_name": "delete_state",
                    "conditions": {
                        "min_index_age": "60d"
                    }
                }
            ]
        },
        {
            "name": "**delete_state**",
            "actions": [
                {
                    "delete": {}
                }
            ],
            "transitions": []
        }
    ],
    "ism_template": [
        {
            "**index_patterns**": [
                "index_name-*"
            ],
            "priority": 0,
            "last_updated_time": 1650470897111
        }
    ]
}
```

该策略有三种状态:*热状态*、*热状态*和*删除状态*。
每个新索引从默认*状态*开始，即*热状态*。
在 *hot_state* 内部，定义了翻转动作。当满足 *min_doc_count* 条件时执行:当索引包含超过 100 个文档时。
在 *hot_state* 中的所有动作完成后，对转换进行评估。在第一个*热状态*中，我们自动决定转到*热状态*。
此时，新创建的索引(在 *hot_state* 中)是将插入新文档的索引，而之前的索引(在 *warm_state* 中)评估其新分配状态的动作和转换。
在 *warm_state* 中没有定义动作，因此我们直接进入转换。这里，当达到*最小索引年龄*时，也就是从索引创建起经过了 60 天时，我们转到*删除状态*。
一旦进入*删除 _ 状态*，索引被删除。
在 *ism_template* 中，我们定义了索引名称模式，用于识别自动应用策略的索引；因此，该策略将被附加到从 *index_name-** 开始的每个新创建的索引。

###### 
3–创建索引(或重新索引现有的索引)

我们现在可以创建第一个索引。为了自动将策略附加到该索引，其名称必须与策略的 *ism_template* 部分中定义的索引模式相匹配。
一般来说，为了应用翻转，索引应该具有以下名称模式:

。*-\d+$。

我们可以看到名字以 *-some_digits* 结尾很重要。这是因为，在每次翻转时，新(自动)创建的索引将与前一个索引同名，数字部分增加 1。

下面是一个索引创建的示例:

```
PUT /index_name-000001
```

###### 4–创建别名

###### T62

我们现在可以将别名与新索引关联起来。
这在翻转阶段使用，因此**别名必须与索引模板(rollover_alias)** 中的别名相同。

```
POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "index_name-000001", **"alias" : "rollover-alias-name"** } }
    ]
}
```

创建索引时，也可以使用请求正文中的相关参数来指定别名:

```
PUT /index-name-000001
{
  "settings": {
	...
  },
 **"aliases": {
    "rollover-alias-name": {}
  },**
  "mappings": {
    ...
   }
}
```

###### 5–将策略附加到索引(如有必要)

最后一步，我们需要将策略附加到新创建的索引上。
如果策略是在索引之前创建的，这将自动完成，否则，我们需要手动附加策略。
此外，该步骤可以通过索引管理部分在 Kibana 中完成:

T3T5

Kibana →索引管理→索引→从列表中选择索引→应用策略→从列表中选择策略

## 摘要

在这篇博文中，我们已经看到了什么是**索引编写器磁盘问题**以及如何解决它。

我们提出两种解决方案:

1.  1.  **临时解决方案:**这里我们解释如何通过手动移除索引块来删除无用的文档，从而释放空间。这种解决方案不能避免错误再次出现。
    2.  **索引管理解决方案(带翻转):**这里我们说明如何自动管理索引以避免错误再次出现。我们定义了一个管理一组索引(由索引名模式定义)的策略，以便在它们达到一定大小时进行翻转。该策略还将删除旧的索引以释放空间。



感谢阅读，下一篇博文再见！

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们同时为 [Elasticsearch 初学者](https://web.archive.org/web/20221202235827/https://sease.io/training/elasticsearch-trainings)和 [Apache Solr 初学者](https://web.archive.org/web/20221202235827/https://sease.io/training/apache-solr-training)提供培训？我们也提供关于这些主题的咨询，如果你想让你的搜索引擎更上一层楼，请联系我们。

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Elasticsearch 磁盘空间问题和翻转解决方案的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！