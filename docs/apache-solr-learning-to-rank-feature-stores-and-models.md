# Apache Solr 学习对特性存储和模型进行排序

> 原文：<https://web.archive.org/web/sease.io/2021/02/apache-solr-learning-to-rank-feature-stores-and-models.html>

这篇简短的博客文章希望帮助你学会在 Apache Solr 中对特性存储和模型进行排序。在这里，我将为您列出主要的命令，而在下一篇文章中，我将回顾在操作特征库和模型时可能出现的最常见的错误。

## 学习对特征进行分级

###### 目录

功能存储是功能的容器。一个模型只使用一个特征库中的特征。
许多型号都可以使用一个功能商店。
collection 1 系列中的可用功能商店列表可从以下网址获得:

```
https://localhost:8983/solr/collection1/schema/feature-store
```

响应看起来像:

```
{
  "responseHeader": {
       "status": 0,
       "QTime": 160
  },
  "featureStores": [
      **"first_store"**,
      **"second_store"**
  ]
}
```

其中**第一商店**和**第二商店**是*收藏 1* 中的可用特色商店。

###### 上传

为了将一个特性存储上传到一个特定集合的特性存储，我们使用下面的 curl 命令。

对于*系列 1* 系列:

```
curl -XPUT "https://localhost:8983/solr/collection1/schema/feature-store/" --data-
binary "@features.json" -H "Content-type:application/json"
```

其中 *features.json* 是包含特性的文件(我们要上传的特性库)。

例如:

```
[
    {
        **"store": "first_store"**,
        "name": "feature_1",
        "class": "org.apache.solr.ltr.feature.ValueFeature",
        "params": {
            "value": "${feature_1}",
            "required": false
        }
    },
    {
        **"store": "first_store"**,
        "name": "feature_2",
        "class": "org.apache.solr.ltr.feature.ValueFeature",
        "params": {
            "value": "${feature_2}",
            "required": false
        }
    },
....
```

这里的“商店”是功能商店中功能商店的名称。上传后，它将出现在功能存储列表中。

###### 删除

要从*收藏 1* 收藏中删除名为 *first_store* 的特色商店:

```
curl -XDELETE "https://localhost:8983/solr/collection1/schema/feature-store/first_store"
```

## 学习给模型排序

###### 目录

如前所述:
一个**型号**只使用一个特性库中的特性**。
一个**特征库**可以被**很多型号**使用。
在*系列 1* 系列中的可用型号列表位于:**

```
https://localhost:8983/solr/collection1/schema/model-store
```

大概是这样的:

```
{
  "responseHeader":{
    "status":0,
    "QTime":188},
  "models":[{
      **"name":"model_1"**,
      "class":"org.apache.solr.ltr.model.MultipleAdditiveTreesModel",
      "store":"first_store",
      "features":[{
          "name":"feature_1",
          "norm":{"class":"org.apache.solr.ltr.norm.IdentityNormalizer"}},
        {
          "name":"feature_2",
          "norm":{"class":"org.apache.solr.ltr.norm.IdentityNormalizer"}},
....
      },
      {
      **"name":"model_2"**,
      "class":"org.apache.solr.ltr.model.MultipleAdditiveTreesModel",
      "store":"first_store",
      "features":[{
          "name":"feature_1",
          "norm":{"class":"org.apache.solr.ltr.norm.IdentityNormalizer"}},
        {
          "name":"feature_3",
          "norm":{"class":"org.apache.solr.ltr.norm.IdentityNormalizer"}},
....
```

这里的“名称”是模型商店中模型的名称。上传后，它将出现在模型存储列表中。

###### 上传

为了将模型上传到特定集合的模型存储中，我们使用 curl 命令:

```
curl -XPUT "https://localhost:8983/solr/collection1/schema/model-store" --data-binary "@solr-model.json" -H "Content-type:application/json"
```

其中 *solr-model.json* 是包含我们想要上传的模型的文件。



例如:

```
{
    "store": "first_store",
    **"name": "model_1"**,
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

###### 删除

要从*集合 1* 集合中删除名为 *model_1* 的模型:

```
curl -XDELETE "https://localhost:8983/solr/collection1/schema/model-store/model_1"
```

## 重新加载

要重新加载*集合 1* 集合:

```
https://localhost:8983/solr/admin/collections?action=RELOAD&name=collection1&wt=xml
```

###### 警告

在使用这些方法时，我们会遇到一些错误。

不要失去下一个职位，以确保正确使用每一种方法！

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220925171341/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和[学习排名](https://web.archive.org/web/20220925171341/https://sease.io/training/learning-to-rank-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220925171341/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 学习排列特性和模型商店的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！