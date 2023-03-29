# 文档值与存储字段:Apache Solr 特性和性能问题

> 原文：<https://web.archive.org/web/sease.io/2020/03/docvalues-vs-stored-fields-apache-solr-features-and-performance-smackdown.html>

这篇博客文章旨在让读者更好地理解 Apache Solr 中的**文档值**和**存储的**字段，以便进行可以互换使用的操作。

虽然**存储的**字段和**文档值**是为了不同的目的而创建的，但两者都可以用来有效地保存文档字段的值，然后在需要时检索它们。

当您定义 solr 模式时，您可能会问自己是否应该将字段定义为 **DocValues** 、 **Stored** 或两者。为了确定这一点，您需要从功能的角度理解应用程序中字段的用途。
如果两种方法都符合你的需求，你应该明白选择其中一种来代替另一种会如何影响搜索引擎的性能和空间使用。

接下来，我会给出一些工具来分析哪种方法是最好的选择，在哪种情况下是最好的选择。

## 存储字段

![](img/c4a2b3a9e255306a8c4a421140cb1ce4.png)

存储字段的主要用途是在查询时返回字段值。您可以按如下方式指定要存储的字段:

`<fieldname="name" type="string" stored="true"/>`

存储的字段以行的方式组织。这意味着，给定一组字段，对于每个文档，这些字段的值被连接在一行中。然后，这些行按照它们的 Lucene 文档 id 顺序存储在磁盘上。根据为该文档和数据类型定义的字段数量，每行可能具有不同的大小(例如，字符串或文本字段具有可变的大小)。存储指向每一行的指针，以便快速访问它们。

假设您想从 Lucene id **lid** 的文档中检索字段 A:

1.  1.  找到**盖**行的地址。
    2.  读取行中的所有值，直到达到字段 a 的值。
    3.  为字段返回一个值。

第二点在这里至关重要。因为我们只在行的开头有一个指针，所以在最坏的情况下，我们必须读取行中所有存储的值。这意味着返回一行中所有存储的字段的成本应该与只返回一个子集的成本大致相同。这是我们定义 solr 模式时要考虑的一个重要因素。

###### 存储字段空间占用

了解存储字段将占用多少空间是一项困难的任务。事实上，当数据被压缩时，一些因素如数据冗余和分布起着基本的作用。

通过将 LZ4 算法应用于如上所述形成的行来进行压缩。LZ4 是一种非常快速的压缩算法，具有良好的但不是最佳的压缩比。可以指定两种模式:**最佳 _ 压缩**和**最佳 _ 速度**。虽然 **BEST_SPEED** 是默认选项，但是为了设置 **BEST_COMPRESSION** ，您只需要在 solrconfig.xml 中启用它，如下所示:

`<codecFactory>
 ` `<str name="compressionMode">BEST_COMPRESSION</str>
</codecFactory>`

该选项仅改变 LZ4 算法执行的一些参数，降低了性能并提高了压缩比。
关于存储字段压缩，最后要说的是，在同一行中，我们可以找到非常异构的数据，这不是获得最佳压缩比的最佳方案。

## 文档值

![](img/980d8cbc855a01dce3e9b757ee072570.png)

**引入了文档值**来提高一些操作的性能，否则这些操作的成本会非常高:**排序**、**分面**和**分组。**如果这些操作在你的搜索引擎用例中频繁且重要，你真的应该考虑将涉及的字段设置为**文档值**。

要启用 **DocValues** ，只需设置 DocValues 属性如下:

`<fieldname="name" type="string" docValues="true"/>`

**DocValues** 可以用来代替存储的字段，以便在查询时检索字段值，但是它们是以列的方式序列化的。需要考虑 3 种重要的行为差异:

1.  1.  文本字段不能设置为**文档值**。如果您的字段具有文本类型，并且您想要返回它，您必须将其设置为存储的。
    2.  **多值字段**使用**不同的** **顺序**返回:当您使用存储字段时，它们以它们被索引的顺序返回(持有数据结构充当**列表**)。相反，内部启用了 docValues 的多值字段使用一个**集合** (SORTED_SET 或 SORTED_NUMERIC，取决于它们的类型)，这是一个数据结构，**对其成员**进行预先排序，**删除重复项**。
    3.  如果您想要返回 fl=*的 **docvalues** ，您需要在字段定义中指定它:**usedocvaluessastored = " true "**(注意这是从模式版本 1.6 开始的默认值)

正如预期的那样，字段是按列序列化的。因此，以通用字段 A 为例，对于集合中的每个文档，我们根据 Lucene 文档 id 连续存储字段 A 的值。请注意，对于某些类型(例如字符串)，每个值都有不同的大小，并且无法预测每个元素需要多少空间。

访问文档 X 的字段 A 的值可以如下进行:

1.  1.  在字段 a 的列中找到文档 X 的地址。
    2.  读取值并返回它。

找到正确的地址实际上是一个棘手的操作，原因有二:

*   *   对于某些文档，该字段可能为空
    *   我们不希望每个**文档值**都有一个指针，否则太多的空间将只用于指针。

然而，因为我们可以直接读取感兴趣的值，我们可以想象使用 **docvalues** 只读取一个字段值比使用存储的字段更有性能，而读取更多的字段需要重复相同的过程更多次。此外，因为值在内存中很稀疏，所以读取更多的值会导致缓存未命中，这会降低性能。

###### 文档值空间占用

不深究细节，花一些篇幅介绍一下文档的空间占用是值得的。这里的事情比存储字段要复杂一些。根据列的不同，数据以不同的方式存储和压缩。因此，包含数字数据的列的存储和压缩方式不同于包含字符串的列。

*   *   数据以列的方式存储，因此同质数据将彼此接近
    *   根据数据类型应用专门的压缩。

由于这些断言，我们可以得出结论，存储 **docvalues** 将导致比使用存储字段更有效的空间使用，占用更少的空间。

## docValues=true 和 stored=true
会发生什么？

有人可能想知道:在检索时 Solr 中会发生什么

1.  1.  fl 仅包含存储值
    2.  fl 仅包含文档值(stored=false)
    3.  fl 包含 stored 和 docValues 字段的混合？

由于这两个选项**不是排他的**，在上面的前两个例子中，我们假设一个选项被启用，另一个被禁用。在这种假设下，情况 1 和 2 很简单，因为 stored 和 docValues 字段是两个不相交集合:在第一种情况下，Solr 将使用存储的值，而在第二种情况下，将使用 docValues 结构。

第三种情况有点棘手，因为如上所述，这两个选项都可以启用(即 stored="true" docValues="true ")。给你。我们必须在 Solr 7 之前和之后做一个区分，因为已经引入了一个优化。

实际上，与优化本身相比，Solr 7 中新的 *RetrieveFieldOptimizer* 类创建了一个集中的责任点，未来可以在这里实现进一步的增强。

在 Solr 7 之前，对于要在结果中返回的每个 *SolrDocument* 实例，实际负责获取字段值的*org . Apache . Solr . response . docsstreamer*类会执行

*   *   存储的字段(即标记为“已存储”的字段，无论其文档值设置如何)
    *   启用文档值的字段

这是那堂课的一个片段:

```
SolrDocument sdoc = null;
...
Document doc = docFetcher.doc(id, fnames);
sdoc = convertLuceneDocToSolrDoc(doc, rctx.getSearcher().getSchema()); 

// decorate the document with non-stored docValues fields
if (dvFieldsToReturn != null) {
  docFetcher.decorateDocValueFields(sdoc, id, dvFieldsToReturn);
}
```

相同的代码在 Solr 7 中看起来“相似”:

```
if (optimizer.returnStoredFields()) {
  Document doc = docFetcher.doc(id, optimizer.getStoredFields());
  // make sure to use the schema from the searcher and not the request (cross-core)
  sdoc = convertLuceneDocToSolrDoc(doc, rctx.getSearcher().getSchema());
} else {
  // no need to get stored fields of the document, see SOLR-5968
  sdoc = new SolrDocument();
}

// decorate the document with non-stored docValues fields
if (optimizer.returnDVFields()) {
  docFetcher.decorateDocValueFields(sdoc, id, optimizer.getDvFields());
}
```

但是正如你所看到的，它引入了一个新的优化器(*retrievefieldsotimizer)*，试图做尽可能最有效的事情。具体来说，如果 fl 仅包含带有以下选项的**字段**

*   *   多值=“假”
    *   docValues="true "

不管它们是否被存储，Solr 都将从 docValues 结构中返回存储的值。然而，如果甚至一个字段是 docValues=false，Solr 将退回到<7 behaviour, that is:

*   *   stored value for stored fields
    *   docValues structure for docValues enabled fields

## Benchmarks

Now that we know our competitors (**存储的** VS **DocValues)，**我们可以探索一个性能对打**。**
我们创建了一个 Solr 实例，旨在提供一些关于**存储的**字段和**文档值**字段返回的性能数据。

这个基准并不意味着是完整的。在某些情况下，事情可能会有所不同。

我们以这种方式创建了一个索引:我们已经索引了来自维基百科的一百万个文档。对于我们添加的每个文档:

*   100 个随机存储的字符串字段，每个字段 15 个字符
*   100 个随机的**文档值**字符串字段，每个字段 15 个字符

文档和查询集合取自 quick wit Github[【1】](https://web.archive.org/web/20220929231246/https://github.com/tantivy-search/search-benchmark-game)。我们使用真实的集合来模拟真实的场景。执行细节:

*   CPU: AMD RYZEN 3600
*   内存:32 GB
*   索引大小:9.07 GB

![](img/b43dd83e81fabef7bd7b70db7a56844a.png)

前 100 名

![](img/8237bb2eebb3762c4328ea9194670aa1.png)

前 200 名

可能会注意到，如果使用**文档值**，检索大量字段会导致性能明显下降。相反，(几乎)令人惊讶的是，通过返回少于 20 个字段， **DocValues** 比存储的字段执行得更好，并且随着返回的字段数量的增加，这种差异变得很小。这是因为主存储器中的**文档值**得到了更好的管理。请求 9 个**文档值**字段和 1 个存储字段平均需要 6.86 个查询时间(多于返回 10 个存储字段)。此外，上面的图表显示，查询时间随着返回的字段数线性增加。这使得针对**文档值**使用存储字段的加速变得可预测。

总之，使用 **DocValues** 从性能角度来看有几个好处(分面、排序和分组),如果只使用少量 **DocValues** 字段而不使用 store 字段，它们甚至可以加速字段检索。此外， **DocValues** 可能比存储的字段使用更少的空间。如果用例需要返回大量的字段，那么使用存储字段是可行的方法。

实验可以通过使用 solr 配置和 python 脚本来复制。git 存储库只包含一小部分用于基准测试的数据样本。[【2】](https://web.archive.org/web/20220929231246/https://github.com/SeaseLtd/solr-field-retrieval-benchmark)

// our service

## 不要脸的塞给我们培训和服务！

我有没有提到我们做 [Apache Solr 初学者](https://web.archive.org/web/20220929231246/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220929231246/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训？
我们也提供这些主题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220929231246/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于文档值和存储字段的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！