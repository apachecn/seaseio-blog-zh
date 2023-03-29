# Apache Solr 特性和模型存储的常见错误

> 原文：<https://web.archive.org/web/sease.io/2021/03/common-errors-with-apache-solr-feature-stores-and-models.html>

正如之前关于 [Apache Solr 学习排列特性和模型存储](https://web.archive.org/web/20220929230917/https://sease.io/2021/02/apache-solr-learning-to-rank-feature-stores-and-models)的博文中所见，在 Apache Solr 中，我们可以对特性存储和模型存储进行许多操作。

T3

在这一页中，我将重点介绍在使用这些方法时可能出现的主要错误，以及我们必须意识到的错误。

## 重新加载

可能发生的第一件事是，我们上传了我们的特性存储或模型，但是我们在相应的特性存储或模型存储中看不到新的组件。

对这两个存储所做的每次更改都需要重新加载集合才能看到结果，因此:

| 在**将**特征库**上传到特征库**后，**重新加载集合**以成功应用更改是很重要的。 |

| 在**将**模型**上传**到模型商店后，**为了成功应用更改，重新加载集合**是很重要的。 |

删除特征存储或模型也是如此:

| 在**删除**所需的**特征存储**、**之后，重新加载集合**以成功应用更改是很重要的。 |

| 在**删除**所需的**型号** *、* **后，重新加载集合**以成功应用更改是很重要的。 |

如何重新加载收藏:

```
http://localhost:8983/solr/admin/collections?action=RELOAD&name=newCollection
```

## 重新加载

Apache Solr 模型上传过程中可能出现的一个错误是:

| 异常:模型类型不存在 |

这是因为**我们上传的模型使用了错误或不存在的特征库**。

为了杀鸡儆猴。



假设我们有这个特征库:

```
{
  "responseHeader": {
       "status": 0,
       "QTime": 160
  },
  "featureStores": [
      "first_store",
      "second_store"
  ]
}
```

假设我们正在上传这个模型:

```
{
    "store": "third_store",
    "name": "first_model",
    "class": "org.apache.solr.ltr.model.MultipleAdditiveTreesModel",
    "features": [
        {
            "name": "feature_1"
        },
        {
            "name": "feature_2"
        },
        {
            "name": "feature_3"
        },
....
```

这个过程将引发一个异常，因为模型的“商店”字段与功能商店中的任何商店名称都不匹配。

| 在这种情况下，我们需要**首先加载特征库**，然后加载模型。
**确保模型中的“商店”字段与特征商店中的正确商店“名称”相匹配。** |

## 删除特征时出错

当删除一个特征库时，我们也必须小心。

T50

如果我们删除一个**用过的**特征库，在集合重新加载的时候我们会得到这个错误:

| org . Apache . Solr . common . sol rexception:org . Apache . Solr . common . sol rexception:无法创建 org . Apache . Solr . ltr . store . rest . managed Model store 类型的新的
managed resource/schema/Model-store，原因是:org . Apache . Solr . common . sol rexception:org . Apache . Solr . ltr . Model 异常:模型类型不存在 org . Apache . Solr . ltr . Model . multipleafteremodel |

在 Solr 用户界面中，我们会在页面顶部找到这个横幅:

| org . Apache . Solr . common . sol rexception:org . Apache . Solr . common . sol rexception:未能创建 org . Apache . Solr . ltr . store . rest . managed Model store 类型的新 managed resource/schema/Model-store，原因是:org . Apache . Solr . common . sol rexception:org . Apache . Solr . ltr . Model exception:模型类型不存在 org . Apache . Solr . ltr . Model . multipleaventreesmodel |

这是因为现在使用已删除特征库的模型找不到对该库的引用。

| 最好是先删除模型，然后在必要时删除相应的特征库。
**只有当没有模型使用时，才能删除特征库。** |

## 关注阿帕奇动物园管理员

另一个可能出现的问题与 ZooKeeper 文件限制有关。
从文档中:

[https://Lucene . Apache . org/Solr/guide/8 _ 8/setting-up-an-external-zookeeper-ensemble . html # increasing-the-file-size-limit](https://web.archive.org/web/20220929230917/https://lucene.apache.org/solr/guide/8_8/setting-up-an-external-zookeeper-ensemble.html#increasing-the-file-size-limit)

甚至在重新加载集合之后，您也看不到您上传的特性存储或模型，这种情况确实可能发生。

我们可以看到 ZooKeeper 可以管理的默认文件大小限制是 1MB。这在加载您的学习排名模型时可能太小，因此可能会导致上传问题。如果文件因其大小而无法上传，我们将无法在功能存储/模型存储中看到它。

| 检查要上传到 Solr 的特征/模型文件的尺寸。
**如有必要增加 ZooKeeper 文件大小限制。** |

## JVM 堆内存使用情况

另一件需要控制的事情是 JVM 堆内存的使用。
功能存储和模型管理所需的内存可能很高，我们可以使可用的内存饱和。

在这张图片中，我们可以看到这种行为的一个例子。在 9 点 12 分，我们的记忆达到饱和，直到 9 点 18 分。



![](img/e134777415c3432a097ad32c5cba485c.png)

| **在商店和模型管理期间，检查 JVM 堆内存使用情况**。
**如有必要增加内存大小。** |

## 好消息！

由于 Alessandro Benedetti 的贡献，在下一个 Solr 8.9 版本中，模型上传和功能删除可能出现的错误将变得更加清晰易读！
可以看到 https://issues.apache.org/jira/browse/SOLR-15149[吉拉问题的新变化](https://web.archive.org/web/20220929230917/https://issues.apache.org/jira/browse/SOLR-15149)

这是一个痛苦的故事。但他告诉我们，他是个好人，他是个好人。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做[学习排名](https://web.archive.org/web/20220929230917/https://sease.io/learning-to-rank-training)和 [Apache Solr 初学者](https://web.archive.org/web/20220929230917/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)培训？我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929230917/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 特性和模型库常见错误的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！