# Elasticsearch 神经搜索教程(白金级/企业级)

> 原文：<https://web.archive.org/web/sease.io/2023/03/elasticsearch-neural-search-tutorial-platinum-enterprise.html>

正如在关于 Elasticsearch 中的[神经搜索的博文中已经提到的，Elastic 8.0 允许用户使用自定义或第三方语言模型(在 PyTorch 中开发)直接在 Elasticsearch 中执行推理，但需要白金或企业订阅才能体验完整的机器学习功能。](https://web.archive.org/web/20230316140007/https://sease.io/2023/03/elasticsearch-neural-search-tutorial.html)

如果您有基本(免费和开放)订阅，Elasticsearch 让您能够在有限的时间内尝试自然语言处理任务(带试用)。

这篇博文以一种非常简单的方式解释了**在 Elasticsearch** 中直接实现文本嵌入和矢量搜索所需的所有步骤。

本教程使用了 **cURL 命令**，Kibana 的安装不是强制性的。

## 神经搜索管道

以下只是一个示意图，用于轻松展示如何将矢量搜索集成到 Elasticsearch 中:

![](img/41b84e31fd726f3ce937fc37f7ac417a.png)

您必须在 Elastic 之外执行您的模型开发，然后您可以轻松地导入它；如果你不想担心模型训练，Elasticsearch 提供了从公共库(拥抱脸、PyTorch 等)导入现成模型的可能性。)借助 [Eland](https://web.archive.org/web/20230316140007/https://pypi.org/project/eland/) 库。
您可以接收数据集，并即时应用导入的模型来执行推理和向量化数据。
您可以接受传入的查询(也由模型编码)，然后使用相似性度量(内部利用 HNSW 数据结构)来排列和返回最佳搜索结果，并改善用户的搜索体验。

在这个简短的描述之后，这里是在 Elasticsearch 中实现神经搜索的端到端管道:

1.  [下载 Elasticsearch](#download)
2.  [开始免费试用](#start)
3.  [部署文本嵌入模型](#deploy)
4.  [创建弹性搜索指数](#createindex)
5.  [创建文本嵌入摄取管道](#createtext)
6.  [索引文件](#index)
7.  [利用矢量场进行搜索](#search)

让我们开始探索每一部分吧！

## 1.下载 Elasticsearch

如果你已经阅读了这篇文章的第一部分，你应该已经在你的系统上安装了 Elasticsearch。如果没有，请参考该教程的[下载部分](https://web.archive.org/web/20230316140007/https://sease.io/wp-admin/post.php?post=55826&action=edit)。

## 2.开始免费试用

为了使用 Elasticsearch (es)内置的 NLP 功能，您必须拥有白金或企业许可证，否则，ES 将提供 30 天免费试用[来访问(和探索)所有订阅功能](https://web.archive.org/web/20230316140007/https://www.elastic.co/guide/en/elasticsearch/reference/current/start-trial.html)。

下面是开始试验的命令:

```
curl -XPOST http://localhost:9200/_license/start_trial?acknowledge=true
```

RESPONSE

```
{
	"acknowledged": true,
	"trial_was_started": true,
	"type": "trial"
}
```

## 3.部署文本嵌入模型

必须[导入并部署](https://web.archive.org/web/20230316140007/https://www.elastic.co/guide/en/machine-learning/current/ml-nlp-deploy-models.html)一个经过适当训练的模型，以便在集群中执行文本嵌入任务。

正如在其他教程中一样，我们使用了来自拥抱脸的预训练模型，这种语言模型称为**[all-MiniLM-L6-v2](https://web.archive.org/web/20230316140007/https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)**(*BERT*)，它将句子映射到 384 维的密集向量空间。

在 ES 中导入模型，可以使用 **[Eland 库](https://web.archive.org/web/20230316140007/https://github.com/elastic/eland)**(elastic search 中机器学习的 Python 客户端)。
首先要做的是打开终端并安装 Eland Python 客户端(带有 PyTorch 额外的依赖项):

```
python -m pip install 'eland[pytorch]'
```

安装后，下面是运行`eland_import_hub_model` [脚本](https://web.archive.org/web/20230316140007/https://github.com/elastic/eland/blob/main/bin/eland_import_hub_model)的命令:

REQUEST

```
eland_import_hub_model --url http://localhost:9200/ --hub-model-id sentence-transformers/all-MiniLM-L6-v2 --task-type text_embedding --start
```

此脚本将从拥抱脸模型中心复制一个模型到 Elasticsearch 集群中；事实上，我们已经定义了:
**`--url` :** 你的 Elasticsearch 集群的 URL **`--hub-model-id`:**拥抱脸模型的标识符 **`--task-type`:**NLP 任务的类型，文本嵌入在我们的案例中 **`--start`**:elastic search 将模型部署到所有可用的机器学习节点，并将模型加载到内存中

RESPONSE

```
INFO : Establishing connection to Elasticsearch
INFO : Connected to cluster named 'elasticsearch' (version: 8.5.3)
INFO : Loading HuggingFace transformer tokenizer and model 'sentence-transformers/all-MiniLM-L6-v2'
INFO : Creating model with id 'sentence-transformers__all-minilm-l6-v2'
INFO : Uploading model definition
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████| 22/22 [00:04<00:00,  5.34 parts/s]
INFO : Uploading model vocabulary
INFO : Starting model deployment
INFO : Model successfully imported with id 'sentence-transformers__all-minilm-l6-v2'
```

从响应中，您可以很容易地看到模型被成功导入，但是您也可以使用这个 API 来检查模型统计信息:

```
curl -XGET http://localhost:9200/_ml/trained_models/_stats
```

RESPONSE

```
{
	"count": 2,
	"trained_model_stats": [{
		"model_id": "lang_ident_model_1",
		"model_size_stats": {
			"model_size_bytes": 1053992,
			"required_native_memory_bytes": 0
		},
		"pipeline_count": 0
	}, {
		**"model_id": "sentence-transformers__all-minilm-l6-v2"**,
		"model_size_stats": {
			"model_size_bytes": 90303761,
			"required_native_memory_bytes": 432265762
		},
		...,
				**"routing_state": {
					"routing_state": "started"**
				},
                                ...
			}]
		}
	}]
}
```

`lang_ident_model_1`是一个内置模型(用于执行语言识别)，已经在集群中提供。

为了便于阅读，我们减少了“我们的”模型的响应主体，但是如果您对此感兴趣[在这里](https://web.archive.org/web/20230316140007/https://www.elastic.co/guide/en/elasticsearch/reference/8.5/get-trained-models-stats.html)您可以找到更多细节。
`"routing_state": "started"`表示模型被分配，准备接受推理请求。

#### 其他请求:

向 **开始** 一个训练有素的模型部署:

```
curl -XPOST http://localhost:9200/_ml/trained_models/sentence-transformers__all-minilm-l6-v2/deployment/**_start**
```

到**停止**到一个训练好的模型部署:

```
curl -XPOST http://localhost:9200/_ml/trained_models/sentence-transformers__all-minilm-l6-v2/deployment/**_stop**
```

对 **删除** 一个训练好的模型:

```
curl -XDELETE http://localhost:9200/_ml/trained_models/sentence-transformers__all-minilm-l6-v2
```

#### 替代方式:

用 Eland 在 Elasticsearch 中导入模型的另一种方法是使用下面的 Python 脚本(也可以在我们的 [GitHub 项目](https://web.archive.org/web/20230316140007/https://github.com/SeaseLtd/vector-search-elastic-tutorial/blob/main/nlp_models/import_model.py)中找到):

```
import elasticsearch
from pathlib import Path
from eland.ml.pytorch import PyTorchModel
from eland.ml.pytorch.transformers import TransformerModel

# Elastic configuration.
ELASTIC_ADDRESS = "http://localhost:9200"

def main():
        # Load a Hugging Face transformers model directly from the model hub
        tm = TransformerModel("sentence-transformers/all-MiniLM-L6-v2", "text_embedding")

        # Export the model in a TorchScript representation which Elasticsearch uses
        tmp_path = "models"
        Path(tmp_path).mkdir(parents=True, exist_ok=True)
        model_path, config, vocab_path = tm.save(tmp_path)

        # Import model into Elasticsearch
        client = elasticsearch.Elasticsearch(hosts=[ELASTIC_ADDRESS])
        ptm = PyTorchModel(client, tm.elasticsearch_model_id())
        ptm.import_model(model_path=model_path, config_path=None, vocab_path=vocab_path, config=config)

if __name__ == "__main__":
    main()
```

我们使用以下命令执行该脚本:

```
python import_model.py
```

请记住，在这一步之后，模型被导入到 Elasticsearch 中，但是您需要手动启动部署以便使用它。

## 4.创建弹性搜索索引

为了创建和定义目标索引的显式映射，可以使用*索引 API*；在本教程中，创建了 **neural_index** :

```
curl http://localhost:9200/**neural_index** -XPUT -H 'Content-Type: application/json' -d '
{
  "mappings": {
    "properties": {
      **"general_text_vector.predicted_value": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine"**
      },
      "general_text": {
        "type": "text"
      },
      "color": {
        "type": "text"
      }
    }
  }
}'
```

按照我们的映射中的定义，文档由 3 个简单的字段组成:

1.  `**general_text_vector.predicted_value**` ( *dense_vector* )将存储摄取管道

    生成的嵌入
2.  将文档`**general_text**` ( *文本*)，源字段与文本一起转换成矢量

3.  **颜色** ( *文本*)，一个额外的字段，只是用来显示过滤查询行为

关于`[dense_vector](https://web.archive.org/web/20230316140007/https://www.elastic.co/guide/en/elasticsearch/reference/8.5/dense-vector.html)`字段类型参数的解释已经在第一篇博文中解决了，所以请参考 [3。为教程的矢量搜索](https://web.archive.org/web/20230316140007/https://sease.io/wp-admin/post.php?post=55826&action=edit)创建一个弹性搜索索引。

## 5.创建文本嵌入摄取管道

可以在 Elasticsearch 中处理数据并自动从文本创建向量，定义一个文本嵌入[摄取管道](https://web.archive.org/web/20230316140007/https://www.elastic.co/guide/en/machine-learning/current/ml-nlp-inference.html#ml-nlp-inference-processor)。
使用以下命令，您可以创建**文本嵌入**管道:

```
curl http://localhost:9200/**_ingest/pipeline/text-embeddings** -XPUT -H 'Content-Type: application/json' -d '
{
  "description": "Text embedding pipeline",
  **"processors": [
    {
      "inference": {
        "model_id": "sentence-transformers__all-minilm-l6-v2",
        "target_field": "general_text_vector",
        "field_map": {
          "general_text": "text_field"
        }
      }
    }
  ]**,
  "on_failure": [
    {
      "set": {
        "description": "Index document to **'\''**failed-<index>**'\''**",
        "field": "_index",
        "value": "failed-{{{_index}}}"
      }
    },
    {
      "set": {
        "description": "Set error message",
        "field": "ingest.failure",
        "value": "{{_ingest.on_failure_message}}"
      }
    }
  ]
}'
```

将为每个段落添加嵌入的[推理处理器](https://web.archive.org/web/20230316140007/https://www.elastic.co/guide/en/elasticsearch/reference/current/inference-processor.html)被指定为:
**`model_id`** :已训练模型的 ID(或别名)
`**target_field**:`将包含结果对象(嵌入)
**`field_map`** 的字段:定义为将`general_text`(段落/文档所在的位置)映射到模型期望的字段`text_field`

我们还添加了一个额外的参数(`on_failure` handler)来处理不同索引中的异常和索引失败，名为`failed-neural_index`。

## 6.索引文档

一旦我们创建了索引和接收管道，我们就可以推送一些文档了。

这里是批量索引请求，其中必须指定 **`pipeline`查询参数**，以便使用我们的“文本嵌入”管道:

```
curl http://localhost:9200/neural_index/**_bulk**?pipeline=text-embeddings**** -XPOST -H 'Content-Type: application/json' -d '
{"index": {"_id": "0"}}
{"general_text": "The presence of communication amid scientific minds was equally important to the success of the Manhattan Project as scientific intellect was. The only cloud hanging over the impressive achievement of the atomic researchers and engineers is what their success truly meant; hundreds of thousands of innocent lives obliterated.", "color": "red"}
'
```

[在这里](https://web.archive.org/web/20230316140007/https://github.com/SeaseLtd/vector-search-elastic-tutorial/blob/main/indexing_phase/create_body_for_bulk.py)您可以找到如何自动创建批量 API 请求的主体。

在上面的简单例子中，只有一个文档被索引。
正如在第一个教程中已经建议的，对于索引许多文档，本地批量 API 可能是有问题的并且非常低效；我们建议使用 [bulk helper](https://web.archive.org/web/20230316140007/https://elasticsearch-py.readthedocs.io/en/v8.5.3/helpers.html) (Pyhton ES 客户端)和以下定制脚本，您可以从一个文件中一次索引一批文档，并使用摄取管道将数据转换为向量:

```
import sys
import time
import random
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk

BATCH_SIZE = 1000

# Elastic configuration.
ELASTIC_ADDRESS = "http://localhost:9200"
INDEX_NAME = "neural_index"
**PIPELINE_NAME = "text-embeddings"**

def index_documents(documents_filename, client):
    # Open the file containing text.
    with open(documents_filename, "r") as documents_file:
            documents = []
            # For each document creates a JSON document including text (and id).
            for index, document in enumerate(documents_file):
                # Generate color value randomly (additional feature to show FILTER query behaviour).
                color = random.choice(['red', 'green', 'white', 'black'])
                # Create the JSON document including index name and pipeline.
                **doc = {
                    "_index": INDEX_NAME,
                    "pipeline": PIPELINE_NAME,
                    "_id": str(index),
                    "general_text": document,
                    "color": color,
                }**
                # Append JSON document to a list.
                documents.append(doc)

                # To index batches of documents at a time.
                if index % BATCH_SIZE == 0 and index != 0:
                    # How you'd index data to Elastic.
                    indexing = bulk(client, documents)
                    documents = []
                    print("Success - %s , Failed - %s" % (indexing[0], len(indexing[1])))
            # To index the rest, when 'documents' list < BATCH_SIZE.
            if documents:
                bulk(client, documents)
            print("Finished")

def main():
    document_filename = sys.argv[1]

    # Declare a client instance of the Python Elasticsearch library.
    client = Elasticsearch(hosts=[ELASTIC_ADDRESS])

    initial_time = time.time()
    index_documents(document_filename, client)
    finish_time = time.time()
    print('Documents indexed in {:f} seconds\n'.format(finish_time - initial_time))

if __name__ == "__main__":
    main()
```

您也可以在我们的 [GitHub 项目](https://web.archive.org/web/20230316140007/https://github.com/SeaseLtd/vector-search-elastic-tutorial/blob/main/indexing_phase/indexer_elastic_with_pipeline.py)中找到它，并且您可以使用以下命令执行该脚本:

```
python indexer_elastic_with_pipeline.py "../from_text_to_vectors/example_input/documents_10k.tsv"
```

#### 重新索引

如果数据已经被索引，您还可以在 reindex 请求中传递'`pipeline`'参数:

```
curl http://localhost:9200/**_reindex** -XPOST -H 'Content-Type: application/json' -d '
{
  "source": {
    "index": "general_index"
  },
  "dest": {
    "index": "neural_index",
    **"pipeline": "text-embeddings"**
  }
}'
```

当前索引(*源*)中的所有文档被复制到具有新映射的新索引(*目标*)中(之前已创建)，摄取管道功能用于为每个文档添加嵌入。

## 7.利用向量场的搜索

当前不支持在搜索请求期间从查询词隐式生成矢量嵌入。

因此，您可以使用模型的 **_infer API** ，将查询转换成向量:

```
curl http://localhost:9200/_ml/trained_models/sentence-transformers__all-minilm-l6-v2/deployment/_infer -XPOST -H 'Content-Type: application/json' -d '{
  "docs": {
    "text_field": "what is a bank transit number"
  }
}'
```

response

```
{"predicted_value":[-0.009013667702674866,-0.07266351580619812,-0.01738189533352852,..., -0.11632353067398071]}
```

为了便于阅读，我们通过插入点来缩短响应。

在获得作为文本查询“什么是银行转帐号”的密集向量的嵌入之后，您可以复制并使用在 kNN 查询中获得的向量。

###### 近似 kNN 示例

```
curl http://localhost:9200/neural_index/_search -XPOST -H 'Content-Type: application/json' -d '{
**"knn": {
    "field": "**general_text_vector.predicted_value**",
    "query_vector": [-9.01364535e-03, -7.26634488e-02, ..., -1.16323479e-01],
    "k": 3,
    "num_candidates": 10
}**,
"_source": [
    "general_text",
    "color"
]}'
```

对于向量相似性搜索，请参考第一篇博文中已经描述的查询( [5。搜索利用矢量场](https://web.archive.org/web/20230316140007/https://sease.io/wp-admin/post.php?post=55826&action=edit))。
唯一的区别是`knn`对象的`field`属性的值:

```
"knn": {
    "field": "**general_text_vector.predicted_value",**
```

代替:

```
"knn": {
    "field": "general_text_vector",
```

## 摘要

我们希望这篇教程能帮助你理解如何使用 cURL 命令代替 Kibana 控制台，在 Elasticsearch 中直接实现文本嵌入和矢量搜索。

这个最近的实现提供了将定制模型集成到 Elasticsearch 中并在内部创建嵌入的能力，与第一篇博客文章中描述的功能相比，这无疑更快且更易于管理，但不幸的是这不是免费的。

// references

NLP:[https://www . elastic . co/guide/en/machine-learning/current/ml-NLP . html](https://web.archive.org/web/20230316140007/https://www.elastic.co/guide/en/machine-learning/current/ml-nlp.html)
矢量搜索:[https://www.elastic.co/what-is/vector-search](https://web.archive.org/web/20230316140007/https://www.elastic.co/what-is/vector-search)
elastic Search 8 中的矢量搜索(Talk):[https://www.youtube.com/watch?v=CM0OSbHTaeA](https://web.archive.org/web/20230316140007/https://www.youtube.com/watch?v=CM0OSbHTaeA)
**博文:**
–[elastic Search 中使用 PyTorch 的现代 NLP 介绍](https://web.archive.org/web/20230316140007/https://www.elastic.co/blog/introduction-to-nlp-with-pytorch-models)
–[如何部署自然语言处理](https://web.archive.org/web/20230316140007/https://www.elastic.co/blog/how-to-deploy-natural-language-processing-nlp-getting-started)
–

// our service

## 还在纠结 Elasticsearch 里的神经搜索？

如果你在 Elasticsearch 中难以实现神经搜索，不要担心——我们会帮助你！
我们的团队提供 **专家服务和培训** 帮助您优化 Elasticsearch 搜索引擎，充分利用您的系统。立即联系我们，了解更多信息！

[Send us an e-mail](https://web.archive.org/web/20230316140007/mailto:info@sease.io)// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于 Elasticsearch 神经搜索教程(白金级/企业级)的帖子吗？不要忘记订阅我们的时事通讯，以便在信息检索世界中保持最新状态！