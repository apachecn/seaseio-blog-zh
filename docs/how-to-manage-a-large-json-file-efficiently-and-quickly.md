# 如何高效快速地管理大型 JSON 文件

> 原文：<https://web.archive.org/web/sease.io/2021/10/how-to-manage-a-large-json-file-efficiently-and-quickly.html>

在这篇博客文章中，我想给你一些提示和技巧来找到用 Python 读取和解析一个大 JSON 文件的有效方法。

**JSON(JavaScript Object Notation)**是一种开放的标准文件格式和数据交换格式，使用人类可读的文本来存储和传输由属性-值对和数组组成的数据对象。

[维基百科](https://web.archive.org/web/20230315122532/https://en.wikipedia.org/wiki/JSON)

通过名为**JSON**[【1】](https://web.archive.org/web/20230315122532/https://docs.python.org/3/library/json.html)的 Python 内置包，处理包含多个 JSON 对象(例如几个 JSON 行)的文件非常简单。一旦导入，这个模块提供了许多方法来帮助我们编码和解码 JSON 数据[【2】](https://web.archive.org/web/20230315122532/https://pynative.com/python-json-load-and-loads-to-parse-json/)。

无论如何，如果您必须解析一个大的 JSON 文件，并且数据的结构太复杂，那么在时间和内存方面会非常昂贵。JSON 通常被整体解析，然后在内存中处理:对于大量数据，这显然是有问题的。

让我们一起来看看一些可以帮助您在 Python 中导入和管理大型 JSON 的解决方案:

###### 1)使用熊猫法。传递 CHUNKSIZE 参数的 READ_JSON

**输入** : JSON 文件
****输出**:熊猫数据帧**

 **' **chunksize** '参数将生成一个读取器，每次读取特定数量的行，并根据文件的长度，创建一定数量的块并将其推入内存，而不是一次读取整个文件；例如，如果您的文件有 100.000 行，并且您传递 chunksize = 10.000，您将获得 10 个块。通过选择数据块的大小，将数据“分割”成更小的块，有望让你将它们放入内存。

**n . b .** –只能与另一个参数成对传递“chunksize”:***lines = True***
–该方法不会返回数据帧，而是返回一个 **JsonReader 对象**进行迭代。**  **###### 2)更改特征的数据类型

Pandas 会自动为我们检测数据类型，但正如我们从文档中所知，默认的数据类型并不是最高效的内存类型。

尤其是对于包含混合数据类型的字符串或列，Pandas 使用 dtype ' **对象**。它占用了大量的内存空间，因此如果可能的话，最好避免使用它。“分类”数据类型当然影响较小，尤其是当与行数相比没有大量可能的值(类别)时。

**pandas.read_json** 方法有一个“**dtype”**参数，使用它可以显式地指定列的类型。它接受以列名作为键、以列类型作为值的字典。

**n . b .**
–如果 orient='table': **orient** 是另一个可以传递给方法以指示预期 JSON 字符串格式的参数，则不能传递' **dtype** '参数。根据官方文档，可以接受许多可能的方向值来指示 JSON 文件的内部结构:split、records、index、columns、values、table。以下是了解 orient 选项并找到适合您情况的选项的参考[【4】](https://web.archive.org/web/20230315122532/https://pandas.pydata.org/docs/user_guide/io.html#reading-json)。请记住，如果使用“table ”,它将遵循 JSON 表模式，允许保留元数据，如 dtype 和索引名，因此不可能传递' **dtype** '参数。

–正如这里所报告的[[5]](https://web.archive.org/web/20230315122532/https://github.com/pandas-dev/pandas/issues/33205),**dtype**参数似乎无法正常工作:事实上，它并不总是应用字典中预期和指定的数据类型。

关于第二点，我给你看一个例子。

我们指定一个字典，并用“dtype”参数传递它:

```
dtypes_dict = {
    'feature1': 'int64',
    'feature2': 'category',
    'feature3': 'float16'
    'feature4': 'int8'
    }

df = pd.read_json('dataset.json', lines=True, dtype=dtypes_dict)
df.info()
```

*的输出**。信息*** 将为:

```
feature1        int64
feature2        object
feature3        float16
feature4        float64
```

你可以看到熊猫忽略了两个特性的设置:

T42

*   *   **feature2** :数据类型是‘object’，而不是字典中规定的‘category’；在这种情况下，将它转换为“类别”的最简单方法是使用**。**astype()[【6】](https://web.archive.org/web/20230315122532/https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html)。让我们使用 *memory_usage()* 来证明使用‘category’而不是‘object’会大大减少内存[【7】](https://web.archive.org/web/20230315122532/https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.memory_usage.html):

```
memory_used_if_object = df['feature2'].memory_usage(deep=True) / 1e6
2.67064

memory_used_if_category = df['feature2'].astype('category').memory_usage(deep=True) / 1e6
0.077111
```

*   *   **特性 4** :数据类型是‘float 64’，不是‘int 8’；很可能是因为存在 ***非有限值(NA 或 inf)*** 而无法将该列转换为整数。

###### 3)删除不重要的列

为了节省更多的时间和内存用于数据操作和计算，您可以简单地删除[【8】](https://web.archive.org/web/20230315122532/https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.drop.html?highlight=drop#pandas.DataFrame.drop)或过滤掉一些您知道在管道开始时没有用的列:

```
df.drop(columns=['feature2', 'feature3'], axis=1, inplace=True)
```

###### 4)使用不同的库:DASK 或 PYSPARK

Pandas 是 Python 编程语言中最流行的数据科学工具之一；它简单、灵活，不需要集群，使复杂算法的实现变得容易，并且对于小数据非常有效。如果您有一定的内存限制，您可以尝试应用上面看到的所有技巧。

尽管如此，在处理大数据时，Pandas 有其局限性，具有并行性和可伸缩性的库可以帮助我们，如 Dask 和 PySpark。

###### **Dask**特点

*   *   开源，包含在 Anaconda 发行版中
    *   熟悉的编码，因为它重用了现有的 Python 库，扩展了 Pandas、NumPy 和 Scikit-Learn 工作流
    *   通过利用多核 CPU 和从磁盘高效传输数据，它可以在单台机器上实现高效的并行计算[【9】](https://web.archive.org/web/20230315122532/https://docs.dask.org/en/latest/why.html)
    *   使用 Dask Bags[【10】](https://web.archive.org/web/20230315122532/https://examples.dask.org/bag.html)或 Dask data frame[【11】](https://web.archive.org/web/20230315122532/https://docs.dask.org/en/latest/dataframe-api.html#dask.dataframe.read_json)的代码实现

###### **PySpark 特性**

*   *   开源，包含在 Anaconda 发行版中
    *   PySpark 的语法和熊猫的很不一样；动机在于 PySpark 是 Apache Spark 的 Python API，用 Scala 编写。为了获得一个熟悉的界面，旨在以最少的努力利用 PySpark，您可以看看由 Databricks 创建的 Pandas API。
    *   像 Dask 一样，它是多线程的，可以利用你机器的所有内核[【13】](https://web.archive.org/web/20230315122532/https://towardsdatascience.com/spark-vs-pandas-part-2-spark-c57f8ea3a781)
    *   代码实现[【14】](https://web.archive.org/web/20230315122532/https://sparkbyexamples.com/pyspark/pyspark-read-json-file-into-dataframe/)

我们还没有尝试过这两个库，但我们很想探索它们，看看它们是否真的像我们在许多文章中读到的那样，是大数据的革命性工具。

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20230315122532/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和[人工智能搜索](https://web.archive.org/web/20230315122532/https://sease.io/artificial-intelligence-in-search-training)培训？
我们也提供这些话题的咨询，[如果你想借助人工智能的力量让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20230315122532/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于如何管理大型 JSON 文件的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！**