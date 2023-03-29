# 弹性搜索别名字段类型

> 原文：<https://web.archive.org/web/sease.io/2020/11/elasticsearch-alias-field-type.html>

Elasticsearch 提供了一个名为 ["alias"](https://web.archive.org/web/20220930003615/https://www.elastic.co/guide/en/elasticsearch/reference/master/alias.html) 的字段类型，可用于定义一个真实字段的附加/替代名称。

为了定义字段别名，您必须提供:

*   *   一个名字
    *   对真实字段的引用



这里有一个例子:

```
{
  "mappings": {
    "properties": {
      "title": {
        "type": "keyword"
      },
      "title_untokenized": {
        "type": "alias",
        "path": "title" 
      }
    }
  }
}
```

可以在不同的上下文中使用该字段来代替目标字段。但是，需要记住一些限制。我们来列举一下。

## 路径必须指示一个存在的，
具体字段

别名**的目标字段不能是**一个**对象**，一个**数组**或另一个**字段别名**，包括字段本身:必须是一个**具体字段**。

此外，它**不能是动态字段**(即在 **dynamic_templates** 部分用通配符声明了名称的字段)，因为**别名绑定**是在**映射时间**完成的。因此，当**别名**是**创建的**时，**目标字段**必须**存在。以下请求:**

```
{
  "mappings": {
    "dynamic_templates": [
      {
        "dates": {
          "match": "*_date",
          "mapping": {
            "type": "date",
            "format": "dd/MM/yyyy"
          }
        }
      }
    ],
    "properties": {
      "title": {
        "type": "text"
      },
      "author": {
        "type": "text"
      },
      "headings": {
        "type": "alias",
        "path": "creation_date"
      }
    }
  }
}
```

将返回 400 错误，并显示以下消息:

###### *字段别名[标题]的[路径]值[创建日期]无效:别名必须引用映射* 中的现有字段

## 别名不能有多个目标字段

这意味着*路径*属性**的值不能**为数组，只能**为单值**。换句话说，不可能有一个字段为多个字段取别名。以下请求:

```
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "author": {
        "type": "text"
      },
      "headings": {
        "type": "alias",
        "path" : ["title", "author"]
      }
    }
  }
}
```

将返回 400 错误，并显示以下消息:

###### *字段别名[标题]的[路径]值[[标题，作者]]无效:别名必须引用映射* 中的现有字段

我同意你的说法，但这并不能说明真正的问题。这是因为路径属性的**整数值被解释为字段名称，而“[[标题，作者]]”并不对应于字段。**

## 目标字段不能是别名

这意味着我们不能在**路径**属性中指出**别名字段**:

```
"headings": {
  "type": "alias",
  "path" : "another_field_whose_type_is_alias"
}
```

类似上面的请求将返回以下错误:

###### *字段别名【标题】的【路径】值【another _ field _ whose _ type _ is _ alias】无效:一个别名不能引用另一个别名*

除了上面的例子，根据同样的规则，我们应该提到:

*   *   别名字段**不能在 **copy_to** 指令中使用**
    *   别名字段**不能用作**多字段**的类型**

上面两点的原因是一样的:别名字段**不存在**，它是**虚拟的**，所以它**不能用作来自另一个字段的值的目的地**，因为它发生在多字段或 copy_to 指令中。



以下定义无效:

```
{
  "properties": {
    "title": {
      "type": "text",
      "copy_to": "alias_field"
    },
    "alias_field": {
      "type": "alias",
      "path": <some other field, even a concrete one>
    }
  }
}
```

```
{
  "properties": {
    "title": { 
      "type": "text",
      "fields": {
        "en": {
          "type": "text",
          "analyzer": "english"
        },
        "it": {
          "type": "alias",
          "path": <some other field, even a concrete one>
        }
      } 
    }
  }
}
```

## (别名)包含点的字段名称

这其实是一个对所有领域都有效的规则。**点字符**在 elasticsearch 中有特殊的含义:它用于定义**派生字段**(称为 **[多字段](https://web.archive.org/web/20220930003615/https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html)** )，这些字段是在**父字段定义**之上创建的，如下例所示:

```
{
  "properties": {
    "title": { 
      "type": "text",
      "fields": {
        "en": {
          "type": "text",
          "analyzer": "english"
        },
        "it": {
          "type": "text",
          "analyzer": "italian"
        }
      } 
    }
  }
}
```

上面的配置将创建三个不同的字段:

*   *   **标题**:父字段。它与一个**标准**分析仪相关联
    *   **title.en** :继承父字段值的派生字段。该值将使用对称(即索引和查询时间)**英语**分析器进行处理
    *   **title.it** :继承父字段值的派生字段。该值将使用对称(即索引和查询时间)**意大利语**分析器进行处理

因此，在上面的定义中，分配给父字段的值也将用于创建派生字段。

字段名可以包含点吗？答案是“看情况”。

具体来说，如果点之前的**部分表示另一个现有字段**，那么答案是**负**，因为 Elasticsearch 会将其解释为去接触的多字段。以下请求:

```
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "title.en": {
        "type": "alias",
        "path": "title" 
      }
    }
  }
}
```

生成 400 响应，并显示以下错误消息:

###### *不能将一个非对象映射[标题]与一个对象映射[标题]* 合并

然而，下面的**请求是完全有效的**:

```
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "author.en": {
        "type": "alias",
        "path": "title"
      }
    }
  }
}
```

它将创建以下映射:

```
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "author.en": {
        "type": "alias",
        "path": "title"
      }
    }
  }
}
```

## 别名不能用于输入文档
(即索引)

由于**是虚拟字段**，输入文档(即发送到 Elasticsearch 进行索引的文档)**不能** **包含别名字段**:它们必须只指示具体字段。

## 源过滤中不能使用别名

记住:别名字段是虚拟的，它们不是源文档的一部分，因此:

*   *   它们**不会在响应中的 **_source** 元素中出现**
    *   它们**不能用于 **[源过滤子句](https://web.archive.org/web/20220930003615/https://www.elastic.co/guide/en/elasticsearch/reference/7.9/search-fields.html)****

## 无支持的

在一些 API 中不能使用别名字段，例如[术语向量](https://web.archive.org/web/20220930003615/https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html)和一些查询，例如[术语](https://web.archive.org/web/20220930003615/https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) / [术语](https://web.archive.org/web/20220930003615/https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html)和 [more_like_this](https://web.archive.org/web/20220930003615/https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html) 。

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Elasticsearch 初学者](https://web.archive.org/web/20220930003615/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)和 [Apache Solr 初学者](https://web.archive.org/web/20220930003615/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220930003615/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于弹性搜索别名字段类型的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！