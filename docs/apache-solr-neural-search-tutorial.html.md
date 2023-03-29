# Apache Solr 神经搜索教程

> 原文：<https://web.archive.org/web/sease.io/2023/01/apache-solr-neural-search-tutorial.html>

嗨读者们！在这篇博文中，我们将探索我们对 Apache Solr 的神经搜索贡献，通过一个端到端的教程提供已经可用的详细描述。

为了更好地理解基于向量的方法是如何改进搜索的，并了解更多关于 Apache Lucene/Solr 实现的信息，我们建议从阅读我们之前的两篇博文开始:

*   *   [Apache Solr 神经搜索](https://web.archive.org/web/20230202201702/https://sease.io/2022/01/apache-solr-neural-search.html)

*   *   [Apache Solr 神经搜索 Knn 基准](https://web.archive.org/web/20230202201702/https://sease.io/2022/01/apache-solr-neural-search-knn-benchmark.html)



这篇文章的目的不是深入实现细节，而是在实践中展示如何使用这个新的 Apache Solr 特性来索引和搜索向量，然后运行完整的端到端神经搜索。
通过实际例子，我们将了解如何:

1.  **Apache Solr 实现工作于** ，引入了新的字段类型和查询解析器
2.  **从文本中生成向量**并将大型语言模型与 Apache Solr 集成
3.  **运行 KNN 查询**(带和不带过滤器)以及如何使用它们进行重新排序



## 神经搜索管道

让我们从使用 Solr 实现神经搜索的端到端管道的概述开始:

1.  [下载 Apache Solr](#download)

2.  [对外产生矢量](#produce)

3.  [创建一个包含向量字段](#create) 的索引
4.  [索引文件](#index)

5.  [利用矢量场进行搜索](#search)

我们现在详细描述每个部分，以便您可以轻松地重现本教程。

#### 1.下载 Solr

神经搜索已于 2022 年 5 月随 Apache Solr 9.0 发布。
本教程使用最新版本(9.1)，可以从:[https://solr.apache.org/downloads.html](https://web.archive.org/web/20230202201702/https://solr.apache.org/downloads.html)下载

Solr 可以安装在任何支持 Java 运行时环境(JRE)版本 11 或更高版本的系统上(Linux、macOS 和 Windows)。

*看一下[指令](https://web.archive.org/web/20230202201702/https://www.apache.org/dyn/closer.cgi#verify)用于验证下载文件的完整性，包括 sha512 密钥和 PGP 密钥。*

将下载的文件解压缩到您想要使用它的位置，从该文件夹打开终端并在本地运行 Solr:

```
bin/solr start
```

您现在可以导航到 Solr 管理界面:[http://localhost:8983/Solr/](https://web.archive.org/web/20230202201702/http://localhost:8983/solr/)

#### 2.外部产生矢量

为了执行利用矢量嵌入的搜索，有必要:

1.  在 Solr 外训练一个模型。
2.  使用自定义脚本从文档字段创建矢量嵌入。
3.  把向量推到 Solr。

对于本教程，我们使用一个 Python 项目，您可以很容易地从我们的 [GitHub](https://web.archive.org/web/20230202201702/https://github.com/SeaseLtd/approaching-neural-search) 页面克隆它。

###### Python 要求

要复制此练习，您只需在 python 环境中安装以下要求:

```
python==3.8.0
sentence-transformers
pysolr
```

###### 自然语言处理模型和语料库

为了将文本编码到相应的向量中，我们没有训练模型，而是使用了名为**[all-MiniLM-L6-v2](https://web.archive.org/web/20230202201702/https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)**的预训练(和微调)模型，这是一种自然语言处理(NLP)句子转换模型。
型号类型为 **BERT** ， *hidden_size* (所以 *`embedding_dimension`* )为 384，大致为 80MB。

对于本教程，我们采用了 MARCO 女士的一个语料库，这是一个用于深度学习的大规模信息检索数据集的集合。特别是，我们下载了文章检索集:[collection.tar.gz](https://web.archive.org/web/20230202201702/https://msmarco.blob.core.windows.net/msmarcoranking/collection.tar.gz)，并索引了大约 10k 篇文章。

###### 创建矢量嵌入

下面是为了从语料库中自动创建矢量嵌入而运行的 python 脚本:

```
from sentence_transformers import SentenceTransformer
import torch
import sys
from itertools import islice

BATCH_SIZE = 100
INFO_UPDATE_FACTOR = 1
MODEL_NAME = 'all-MiniLM-L6-v2'

# Load or create a SentenceTransformer model.
model = SentenceTransformer(MODEL_NAME)
# Get device like 'cuda'/'cpu' that should be used for computation.
if torch.cuda.is_available():
    model = model.to(torch.device("cuda"))
print(model.device)

def batch_encode_to_vectors(input_filename, output_filename):
    # Open the file containing text.
    with open(input_filename, 'r') as documents_file:
        # Open the file in which the vectors will be saved.
        with open(output_filename, 'w+') as out:
            processed = 0
            # Processing 100 documents at a time.
            for n_lines in iter(lambda: tuple(islice
            (documents_file, BATCH_SIZE)), ()):
                processed += 1
                if processed % INFO_UPDATE_FACTOR == 0:
                    print("processed {} batch of documents"
                    .format(processed))
                # Create sentence embedding
                vectors = encode(n_lines)
                # Write each vector into the output file.
                for v in vectors:
                    out.write(','.join([str(i) for i in v]))
                    out.write('\n')

def encode(documents):
    embeddings = model.encode(documents, show_progress_bar=True)
    print('vector dimension: ' + str(len(embeddings[0])))
    return embeddings

def main():
    input_filename = sys.argv[1]
    output_filename = sys.argv[2]
    batch_encode_to_vectors(input_filename, output_filename)

if __name__ == "__main__":
        main()
```

RESPONSE

```
processed 1 batch of documents
Batches: 100%|██████████| 4/4 [00:04<00:00,  1.08s/it]
vector dimension: 384
...
...
processed 100 batch of documents
Batches: 100%|██████████| 4/4 [00:02<00:00,  1.35it/s]
```

是一个 Python 框架，你可以用它来计算句子/文本的嵌入；它提供了针对各种任务调整的大量[预训练模型](https://web.archive.org/web/20230202201702/https://www.sbert.net/docs/pretrained_models.html)，在这种情况下，我们使用 **all-MiniLM-L6-v2** 将句子映射到 384 维密集向量空间。

python 脚本将包含 10k 个文档的文件(即 MS MARCO passage 检索集合的一小部分)作为输入:

```
sys.argv[1] = "/path/to/documents_10k.tsv"
```

*例如 1 份文件*

*科学头脑中的交流对于曼哈顿计划的成功与科学智慧同样重要。笼罩在原子研究人员和工程师令人印象深刻的成就上的唯一乌云是他们的成功真正意味着什么；数十万无辜的生命被抹杀。*

它将输出一个包含相应向量的文件:

```
sys.argv[2] = "/path/to/vectors_documents_10k.tsv"

```

*如 1 号文件*

```
0.0367823,0.072423555,0.04770486,0.034890372,0.061810732,0.002282318
,0.05258357,0.013747136,-0.0060595,...,0.0054274425
```

有必要将获得的嵌入推送到 Solr(我们将在索引文档一节中看到这一点)。

// are you finding the tutorial a bit difficult? No worries!

#### 获取我们关于 Apache Solr 神经搜索的实时教程的门票

你可以向我们的神经搜索专家现场提问 [Alessandro Benedetti](https://web.archive.org/web/20230202201702/https://sease.io/about/alessandro-benedetti) 和 [Ilaria Petreti](https://web.archive.org/web/20230202201702/https://sease.io/ilaria-petreti)

[LIVE TUTORIAL
APACHE SOLR
NEURAL SEARCH](https://web.archive.org/web/20230202201702/https://sease.io/information-retrieval-mini-training-2/end-to-end-apachesolr-neural-search-tutorial) [LIVE TUTORIAL - APACHE SOLR NEURAL SEARCH](https://web.archive.org/web/20230202201702/https://sease.io/information-retrieval-mini-training-2/end-to-end-apachesolr-neural-search-tutorial)

#### 3.创建包含向量字段的索引

在安装和启动 Solr 之后，要做的第一件事是创建一个**集合**(即一个单独的索引和相关的事务日志和配置文件)，以便能够进行索引和搜索。

下面是创建' ***ms-marco*** 集合的命令:

```
bin/solr create -c ms-marco

```

为了使本教程尽可能简单，让我们回顾并编辑配置文件，特别是:

solrconfig.xml

它定义了索引选项、请求处理程序、高亮显示、拼写检查和各种其他配置。

以下是我们的最小配置文件:

```
<?xml version="1.0" ?>
<config>
  <dataDir>${solr.data.dir:}</dataDir>

  <directoryFactory name="DirectoryFactory"           class="${solr.directoryFactory:solr.NRTCachingDirectoryFactory}"/>
  <schemaFactory class="ClassicIndexSchemaFactory"/>

  <luceneMatchVersion>LATEST</luceneMatchVersion>

  <updateHandler class="solr.DirectUpdateHandler2">
    <commitWithin>
      <softCommit>${solr.commitwithin.softcommit:true}</softCommit>
    </commitWithin>
  </updateHandler>

  <requestHandler name="/select" class="solr.SearchHandler">
    <lst name="defaults">
      <str name="echoParams">explicit</str>
      <str name="indent">true</str>
      <str name="df">text</str>
    </lst>
  </requestHandler>
</config>
```

schema.xml

它是允许定义数据模型的入口点，因此要索引的字段和字段的类型(文本、整数等。).

Solr 从启用托管模式开始，但是为了简单和手动编辑文件，我们切换到静态模式。欲了解更多信息，请阅读此处的。

同样，我们尽可能保持它的最小化，只包括必要的字段:



```
<schema name="ms-marco" version="1.0">
  <fieldType name="string" class="solr.StrField" omitNorms="true" positionIncrementGap="0"/>
  <!-- vector-based field -->
  **<fieldType name="knn_vector" class="solr.DenseVectorField" vectorDimension="384" omitNorms="true"/>**
  <fieldType name="long" class="org.apache.solr.schema.LongPointField" docValues="true" omitNorms="true" positionIncrementGap="0"/>
  <!-- basic text field -->
  <fieldType name="text" class="solr.TextField">
    <analyzer>
      <tokenizer class="solr.StandardTokenizerFactory"/>
      <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
  </fieldType>

  <field name="id" type="string" indexed="true" stored="true" multiValued="false" required="false"/>
  <field name="text" type="text" indexed="true" stored="true"/>
  **<field name="vector" type="knn_vector" indexed="true" stored="true" multiValued="false"/>**
  <field name="_version_" type="long" indexed="true" stored="true" multiValued="false"/>
  <uniqueKey>id</uniqueKey>

</schema>
```

模式是告诉 Solr 应该如何从输入文档构建索引的地方。它用于配置字段，指定一组字段属性来控制将要产生的数据结构。

按照我们的模式定义，文档由 3 个简单的字段组成:

1.  **`id`**
2.  `the document `**text**`(将带有文本的源字段转换成矢量)`
`*   **`vector`** 存储由前面章节中看到的 Python 脚本生成的嵌入`

 `目前，密集矢量场不支持*文档值*和*多值*。

**密集向量字段** [ [1](https://web.archive.org/web/20230202201702/https://solr.apache.org/guide/solr/latest/query-guide/dense-vector-search.html) ]提供了索引和搜索浮动元素密集向量的可能性。在这种情况下，我们将其定义为:

*   **名称**:字段类型名称
*   **等级** : solr。DenseVectorField
*   **vectorDimension** :要传入的密集向量的维度，需要等于模型维度。在这种情况下 *384* 。

***当前** **限制** :
向量的最大基数当前被限制为 **1024** ，除了出于性能考虑之外没有其他特殊原因；将来可能会增加，但就目前而言，如果您想使用更大的向量大小，您需要自定义 Lucene build，然后在 Solr 中设置它。*

我们保留了其他参数的默认值，特别是:

*   **similarityFunction** :向量相似度函数，用于将前 K 个最相似的向量返回给目标向量。默认为**欧几里德**，否则可以使用**点积**或者**余弦**。

*   **knnAlgorithm** :要使用的底层 *knn* 算法； **hnsw** 是目前唯一支持的。
    **–hnswMaxConnections**:控制有多少最近邻候选连接到新节点。默认为 **16** 。
    **–hnswbeamwwidth**:在图中搜索每个新插入的节点时要跟踪的最近邻候选节点的数量。默认为 **100** 。

***hnswMaxConnections***和***hnswbeamwwidth***是高级参数，与当前使用的算法严格相关；它们影响索引时构建图形的方式，所以除非您真的需要它们并且知道它们的影响，否则建议不要更改这些值。在* [Solr 文档](https://web.archive.org/web/20230202201702/https://solr.apache.org/guide/solr/latest/query-guide/dense-vector-search.html) *中可以找到 Solr 参数与* HNSW 2018 论文*参数*的映射关系。*

**N.B.**
修改完所有配置文件后，需要**重新加载集合**(或停止并重启 Solr)。`  `#### 4.索引文档

一旦我们创建了矢量嵌入和索引，我们就可以推送一些文档了。

Solr 中的向量索引相当简单，与多值浮点没有太大区别。

我们使用 [pysolr](https://web.archive.org/web/20230202201702/https://pypi.org/project/pysolr/) ，一个 Apache Solr 的 Python 包装器，来索引成批的文档。
下面是 python 脚本:

```
import sys
import pysolr

## Solr configuration.
SOLR_ADDRESS = 'http://localhost:8983/solr/ms-marco'
# Create a client instance.
solr = pysolr.Solr(SOLR_ADDRESS, always_commit=True)

BATCH_SIZE = 100

def index_documents(documents_filename, embedding_filename):
    # Open the file containing text.
    with open(documents_filename, "r") as documents_file:
        # Open the file containing vectors.
        with open(embedding_filename, "r") as vectors_file:
            documents = []
            # For each document creates a JSON document including 
            both text and related vector. 
            for index, (document, vector_string) in enumerate
            (zip(documents_file, vectors_file)):

                vector = 
                [float(w) for w in vector_string.split(",")]
                doc = {
                    "id": str(index),
                    "text": document,
                    "vector": vector
                }
                # Append JSON document to a list.
                documents.append(doc)

                # To index batches of documents at a time.
                if index % BATCH_SIZE == 0 and index != 0:
                    # How you'd index data to Solr.
                    solr.add(documents)
                    documents = []
                    print("==== indexed {} documents ======"
                    .format(index))
            # To index the rest, when 'documents' list < BATCH_SIZE.
            if documents:
                solr.add(documents)
            print("finished")

def main():
    document_filename = sys.argv[1]
    embedding_filename = sys.argv[2]
    index_documents(document_filename, embedding_filename)

if __name__ == "__main__":
    main()
```

RESPONSE

```
==== indexed 100 documents ======
==== indexed 200 documents ======
...
finished
```

python 脚本将接受两个输入文件，一个包含文本，另一个包含相应的向量:

```
sys.argv[1] = "/path/to/documents_10k.tsv"
sys.argv[2] = "/path/to/vectors_documents_10k.tsv"
```

对于两个文件的每个元素，脚本创建一个 JSON 文档(包括 id、文本和向量)并将其添加到一个列表中；当列表达到 BATCH_SIZE 集合时，JSON 文档将被推送到 Solr。
例如 JSON:

```
{
	'id': '0',
	'text': 'The presence of communication amid scientific minds was equally important to the success of the Manhattan Project as scientific intellect was. The only cloud hanging over the impressive achievement of the atomic researchers and engineers is what their success truly meant; hundreds of thousands of innocent lives obliterated.\n',
	'vector': [0.0367823, 0.072423555, 0.04770486, 0.034890372, 0.061810732, 0.002282318, 0.05258357, 0.013747136, -0.0060595, 0.020382827, 0.022016432, 0.017639274, ..., 0.0054274425]
}
```

在这一步之后，Solr 中已经有 10，000 个文档被索引，我们准备好基于查询检索它们。

// are you finding the tutorial a bit difficult? No worries!

#### 获取我们关于 Apache Solr 神经搜索的实时教程的门票

你将能够向我们的神经搜索专家[亚历山德罗·贝内代蒂](https://web.archive.org/web/20230202201702/https://sease.io/about/alessandro-benedetti)和[伊拉莉亚·彼得雷蒂](https://web.archive.org/web/20230202201702/https://sease.io/ilaria-petreti)现场提问所有问题

[LIVE TUTORIAL
APACHE SOLR
NEURAL SEARCH](https://web.archive.org/web/20230202201702/https://sease.io/information-retrieval-mini-training-2/end-to-end-apachesolr-neural-search-tutorial) [LIVE TUTORIAL - APACHE SOLR NEURAL SEARCH](https://web.archive.org/web/20230202201702/https://sease.io/information-retrieval-mini-training-2/end-to-end-apachesolr-neural-search-tutorial)

#### 5.利用向量场的搜索

为了进行一些查询，我们从 Marco 女士那里下载了段落检索查询:[queries.tar.gz](https://web.archive.org/web/20230202201702/https://msmarco.blob.core.windows.net/msmarcoranking/queries.tar.gz)

以下示例中报告的查询是:`"what is a bank transit number"`。
为了将其转换成向量并在 KNN 查询中使用，我们运行 python 脚本*single-sentence-transformers . py*(来自我们的 GitHub 项目):

```
from sentence_transformers import SentenceTransformer

# The sentence we like to encode.
sentences = ["what is a bank transit number"]

# Load or create a SentenceTransformer model.
model = SentenceTransformer('sentence-transformers/all-MiniLM-L6-v2')

# Compute sentence embeddings.
embeddings = model.encode(sentences)

# Create a list object, comma separated.
vector_embeddings = list(embeddings)
print(vector_embeddings)
```

输出将是一个必须复制到查询中的浮点数组:

```
**[-9.01364535e-03, -7.26634488e-02, -1.73818860e-02, ..., ..., -1.16323479e-01]**
```

以下是神经搜索查询的几个例子:

###### KNN 查询

从搜索的角度来看，Solr 引入了一个新的查询解析器: **knn 查询解析器**

它只接受几个参数作为输入:

*   **f** :存储矢量嵌入的字段
*   **topK:** 要检索的最近邻的个数
*   **向量查询**:表示查询的方括号内的浮点值列表

REQUEST

```
curl -X POST http://localhost:8983/solr/ms-marco/select?fl=id,text,score -d '
{
  "query": "**{!knn f=vector topK=3}[-9.01364535e-03, -7.26634488e-02, -1.73818860e-02, ..., -1.16323479e-01]**"
}'
```

**注意:** 这个查询应该是一个 *POST* ，因为这个向量很可能会超过一个 GET URL 所能接受的字符数。为了便于阅读，在报告查询时，我们通过插入点来减少(非常长的)向量的长度。

RESPONSE

```
{
  "responseHeader":{
    ...,
    ...,
  "response":{**"numFound":3**,"start":0,"maxScore":0.44739443,"numFoundExact":true,"docs":[
      {
        "id":"**7686**",
        "text":["A. A federal tax identification number ... to identify your business to several federal agencies responsible for the regulation of business.\n"],
        "score":0.44739443},
      {
        "id":"**7691**",
        "text":["A. A federal tax identification number (also known as an employer identification number or EIN), is a number assigned solely to your business by the IRS.\n"],
        "score":0.44169965},
      {
        "id":"**7692**",
        "text":["Letâs start at the beginning. A tax ID number or employer identification number (EIN) is a number ... to a business, much like a social security number does for a person.\n"],
        "score":0.43761322}]
  }}
```

设置好 *topK=3* 后，我们得到了查询“*什么是银行转帐号码*”的最佳三个文档。

同样，为了便于阅读，我们将查询响应中包含的信息限制为指定的字段列表( **`fl`** 参数)，而不打印(非常长的)密集向量字段。

###### KNN +预过滤

为了克服 Solr 9.0 的[文档中描述的后置过滤的限制，Apache Solr 9.1 引入了**前置过滤**](https://web.archive.org/web/20230202201702/https://solr.apache.org/guide/solr/9_0/query-guide/dense-vector-search.html#usage-with-filter-queries)

这种贡献提供了运行过滤查询的可能性，以便可以首先缩小搜索范围，然后搜索 *topK* 邻居。

主查询中基于向量的 knn 搜索和过滤查询中的经典词汇搜索:

REQUEST

```
curl -X POST http://localhost:8983/solr/ms-marco/select?fl=id,text,score -d '
{
  **"query" : "{!knn f=vector topK=3}[-9.01364535e-03, -7.26634488e-02, -1.73818860e-02, ..., -1.16323479e-01]"**,
  "filter" : "id:(7686 7692 1001 2001)"
}'
```

RESPONSE

```
{
  "responseHeader":{
   ...,
   ...,
  "response":{**"numFound":3**,"start":0,"maxScore":0.44739443,"numFoundExact":true,"docs":[
      {
        "id":"**7686**",
        "text":["A. A federal tax identification number ... of business.\n"],
        "score":0.44739443},
      {
        "id":"**7692**",
        "text":["Letâs start at the beginning. A tax ID number ... for a person.\n"],
        "score":0.43761322},
      {
        "id":"**1001**",
        "text":["The 23,000-square-mile (60,000 km2) Matanuska-Susitna ... in 1818.\n"],
        "score":0.33552456}]
  }}
```

结果通过“*FQ = id:(7686769210012001)*进行预过滤，然后只有来自该子集的文档被认为是 knn 检索的候选。
设置了 *topK=3* 之后，knn query 从 4 个文档的子集中提取了前 3 个文档。

###### 混合搜索

另一个可用的特性是结合混合的密集和稀疏检索。
在 Apache Solr 中，有查询解析器可以让你通过组合不同查询解析器的结果来构建一个查询；比如 **[布尔查询解析器](https://web.archive.org/web/20230202201702/https://solr.apache.org/guide/solr/latest/query-guide/other-parsers.html#boolean-query-parser)** ，在这里你可以定义多个子句，这些子句将影响搜索结果如何从你的索引中匹配以及如何在你的排名中得分。

在下面的例子中，我们定义了 2 个`should`子句:第一个是词法子句，而第二个子句是纯基于向量的搜索子句。

REQUEST

```
curl -X POST http://localhost:8983/solr/ms-marco/select?fl=id,text,score -d '
{
    "query": {
        "bool": {
            "should": [
            "{!type=field f=id v='7686'}",
 **"{!knn f=vector topK=3}[-9.01364535e-03, -7.26634488e-02, -1.73818860e-02, ..., -1.16323479e-01]**"
            ]
        }
    }
}'
```

RESPONSE

```
{
  "responseHeader":{
   ...}},
  "response":{"numFound":3,"start":0,"maxScore":4.4451337,"numFoundExact":true,"docs":[
      {
        "id":"7686",
        "text":["A. A federal tax identification number ... of business.\n"],
        "score":4.4451337},
      {
        "id":"7691",
        "text":["A. A federal tax identification number ... by the IRS.\n"],
        "score":0.44169965},
      {
        "id":"7692",
        "text":["Letâs start at the beginning. A tax ID number ... for a person.\n"],
        "score":0.43761322}]
  }}
```

在这种情况下，搜索结果的最终列表是从两个子句(should)导出的文档的组合，其中 *id:7686* 具有较高的分数，因为通过匹配两个子句，两个分数被加在一起。

###### 重新排序查询

目前可以将[查询重排序](https://web.archive.org/web/20230202201702/https://solr.apache.org/guide/solr/latest/query-guide/query-re-ranking.html)与 knn 查询解析器一起使用。

查询重新排序允许您运行一个简单的查询( *q* )来匹配文档，然后使用一个更复杂的查询(在这种情况下是 knn 查询)返回的分数来重新排序文档。

REQUEST

```
curl --location -g --request POST "http://localhost:8983/solr/ms-marco/select?q=id:(1001 7686 7692 2001)&fl=id,text,score**&rq={!rerank reRankQuery=$rqq reRankDocs=4 reRankWeight=1}&rqq={!knn f=vector topK=4}[-9.01364535e-03, -7.26634488e-02, ..., -1.16323479e-01]"**
```

注意
为了便于阅读，我们已经对这个查询进行了解码，但是要使它工作，您只需对值进行编码(删除空格，例如`q=id:(1001%207686%207692%202001)`，rq= `{!rerank%20reRankQuery%3D%24rqq%20reRankDocs%3D4%20reRankWeight%3D1}`)并禁用 bash 历史扩展。

RESPONSE

```
{
  "responseHeader":{
   ....}},
  "response":{"**numFound":4**,"start":0,"maxScore":3.9977393,"numFoundExact":true,"docs":[
      {
        "id":"**7686**",
        "text":["A. A federal tax identification number ... of business.\n"],
        "**score":4.4451337**},
      {
        "id":"**7692**",
        "text":["Letâs start at the beginning. A tax ID number ... for a person.\n"],
        "**score":4.4353523**},
      {
        "id":"1001",
        "text":["The 23,000-square-mile (60,000 km2) Matanuska-Susitna ... in 1818.\n"],
        "score":3.9977393},
      {
        "id":"2001",
        "text":["The researchers were in Baltimore on Tuesday ... believed.\n"],
        "score":3.9977393}]
  }}
```

目前不推荐使用 knn 查询解析器进行重新排序，原因如下。

这里所发生的是，第二次通过分数(从 reRank 查询中得出)仅针对要搜索的目标向量的 k 个最近邻居内的文档进行计算，并且当前的主要限制是，它是在整个索引(不是从主查询中得出的文档的子集)上计算**。**

事实上，在这种情况下:

*   对于 *id:7686* 和 *id:7692* ，第一次通过分数(从主查询`q`得出)被添加到第二次通过分数，并乘以乘法因子(`reRankWeight` )

*   对于 *id:1001* 和 *id:2001* ，原始文档分数保持不变，因为它们匹配原始查询，但不匹配 knn 重新排序查询(在 *topK* 之外)。

因此，您并没有对每个第一轮检索搜索结果进行一对一的重新评分，而是有效地将它们与 knn 结果相交。

## 未来作品

我们最近的开源贡献最终将[神经搜索引入了 Apache Solr](https://web.archive.org/web/20230202201702/https://sease.io/2022/01/apache-solr-neural-search.html) ，我们希望本教程能够帮助您理解如何利用这一新的 Solr 特性来改善您的搜索体验！

仍然有一些工作要做，特别是，我们计划为 Apache Solr 做出贡献:

*   一个**模型管理工具**，它可以是一个单独的虚拟机，并利用不同的存储系统(而不是 Zookeeper)。这个想法是以类似于学习排序集成的方式来管理语言模型。

*   一个**更新请求处理器**，它接受一个 BERT 模型的输入，并在**索引时间**进行推理矢量化，以自动丰富文档。

*   一个**查询解析器**，它接受一个 BERT 模型的输入，并在**查询时间**进行推理向量化，以便自动对查询进行编码。

// our service

## 不要脸的塞给我们培训和服务！

正如我所说，我们运行一个有用的[端到端 Apache Solr 神经搜索教程。](https://web.archive.org/web/20230202201702/https://sease.io/information-retrieval-mini-training-2/end-to-end-apachesolr-neural-search-tutorial)
但是我们也提供这方面的咨询，所以[联系](https://web.archive.org/web/20230202201702/https://sease.io/contacts)如果你想借助 AI 的力量让你的搜索引擎更上一层楼！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于 Apache Solr 神经搜索教程的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！`