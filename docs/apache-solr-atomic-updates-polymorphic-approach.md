# Apache Solr 原子更新:多态方法

> 原文：<https://web.archive.org/web/sease.io/2020/01/apache-solr-atomic-updates-polymorphic-approach.html>

在这篇文章中，我们描述了一种解决应用程序需要完全更新和原子更新问题的方法，使用了面向对象编程中一个强大的概念:**多态**。

在面向对象编程中，多态性指的是一个变量、方法或对象采取多种形式的能力。

尽管为了提供高水平的视角，已经对示例上下文进行了抽象，但是所描述的方法的实际应用已经在**户外搜索服务**[【1】](https://web.archive.org/web/20220925175711/https://github.com/Alfresco/SearchServices)中实现。

【T6

**Alfresco 搜索服务**通过利用 **Apache Solr** 为 **Alfresco 内容服务**提供搜索功能。企业版和社区版的 Alfresco 内容服务都使用它。

## 语境

**现有代码**从传入的数据模型中创建 *SolrInputDocument* 实例。文档一旦创建，就被发送到 Solr 进行索引。
每个文档代表**一个域对象的完整状态**:也就是说第一次发送的时候会被**插入**；下一次发送相同的文件(即具有相同 id 的文件)时，**会替换**现有文件。

T33

这是系统的核心部分，逻辑相当复杂:在几个地方创建了一个 *SolrInputDocument* 实例，并传递了许多方法，这些方法用一组特定的属性来丰富它。大概是这样的:

```
public void indexScenario1(DomainObject o) {
    SolrInputDocument doc = new SolrInputDocument();

    ...

    addAttributeSetA(doc, Domain);
    addAttributeSetB(doc, Domain);

    if (something) 
       addAttributeSetC(doc, Domain);
    else
       addAttributeSetD(doc, Domain);
    ... 
```

## 挑战

在我们的贡献下，创建领域模型实例的系统部分有了一点改变:主要的改进包括使用**“delta”对象**的额外能力。换句话说，调用者代码能够向该索引组件提供“完整”或“部分”域对象(即，仅包含已更新内容的域对象)。

## 限制

到目前为止，你认为这是一个完美的适合使用**原子更新**！绝对正确:只包含更改位的域对象可以在**partial***SolrInputDocument*实例中转换，然后发送到 Solr 进行索引。

然而，一个**第一约束**需要被解决:部分对象**不会是一个独占场景，我们仍然必须处理完整对象**。

**第二个约束**:如上所述，索引组件代表了系统的中心/关键部分，因此即使是**最小的变更**也会带来一定程度的风险，因此代码变更应该被最小化。

根据我们的经验，这需要一种“改变越少越好”的方法，而旧的好的**面向对象编程**在这方面绝对很棒！

## 什么是原子更新？

原子更新[【2】](https://web.archive.org/web/20220925175711/https://lucene.apache.org/solr/guide/6_6/updating-parts-of-documents.html)是一种使用**“更新”语义**在客户端执行索引命令的方式，通过应用/索引表示域对象的**部分状态的文档。**

因此，实际上，使用**原子更新**，客户端能够**只发送“部分”文档**，该文档只包含需要应用于现有(即先前索引的)文档的更新。

T25T27

让我们看一个例子。索引以下文档后:

```
{
   "id": 1
   "title": "Design Patterns: Elements of Reusable Object-Oriented Software",
   "author": [
      "Erich Gamma",
      "Richard Helm",
      "Ralph Jonson"
   ]
} 
```

你意识到“拉尔夫·约翰逊”中少了一个“h”(啊！误认这样一个古鲁的名字:不可接受！);另外，你忘了约翰·弗利塞斯……真是一场灾难！

所以你可以做下面两件事之一。

通常的方法是**重新创建整个文档**而不出错，并将其重新发送给 Solr:

```
{
  "id":1
  "title":"Design Patterns: Elements of Reusable Object-Oriented Software",
  "author":[
      "Erich Gamma",
      "Richard Helm",
      "Ralph Johnson",
      "John Vlissides"
  ]
}      

```

那个新文档完全用**替换了索引的文档**(注意:隐含的假设是 *uniqueId* 字段是“Id”)。

另一种方式允许我们只发送我们想在现有文档上更改的内容。在这种情况下，我们将向 Solr 发送这样一个文档:

```
{
  "id": 1
  "author": {
     "remove": "Ralph Jonson",
     "add": ["Ralph Johnson", "John Vlissides"]
  }
}    
```

它将把 id=1 的索引文档作为目标

*   它**去掉** s 错误的值(“拉尔夫·琼森”)
*   它为作者添加了正确的值(“拉尔夫·乔”
*   这是另一个失踪的作者

如你所见，需要更新的字段的值不再是文字 **值**(如字符串、整数)或值列表；相反，我们有一个映射图**,其中**键**是我们想要应用的**更新命令**(例如，删除、添加、设置),而**值**是我们想要用于更新的一个或多个文字值**。****

 **关于 **AtomicUpdates** 的完整语义的更多信息可以在 **Apache Solr 参考指南** [2]中找到:这里需要记住的是 Solr 方面，在幕后没有发生“真正的”部分更新:旧版本的文档是**获取的**，它是**合并了**和**部分状态**；之后，新的“完整”结果文档被再次索引**。**

 **尽管如此，它还是非常有益的，因为当你需要更新文档时，它减少了你可能转移到 Solr 的数据量。

在 **Java** 中，特别是在 **SolrJ** 中， *SolrInputDocument* 类表示我们发送给 Solr 用于索引的数据。这基本上是一个映射，所以我们可以添加，设置或删除字段和值。

我们对以下三种方法感兴趣:****  ```
// If a field with that name doesn’t exist it adds a new entry with the 
// corresponding value, otherwise the value is collected together with 
// the existing value(s)
// This is typically used on multivalued fields (i.e. calling twice this
// method on the same field, will collect 2 xvalues for that field)  
addField(String name, Object value)     

// Sets/Replaces a field value
setField(String name, Object value)     

// Remove a field from the document
removeField(String name, Object value)
```

**同类**也用于表示**部分文档**。您可以通过在 *setField* 或 *addField* 方法中设置 map as 值来实现。贴图可以有一个或多个修改器:

*   *   **“add”**:将指定值添加到多值字段中。
    *   **“删除”**:从多值字段中删除指定值的所有匹配项。
    *   **"set"** :用指定的值设置或替换字段值，如果新值指定为' null '或空列表，则删除这些值。



注意还有两个额外的修饰符(inc，removeregex ),但是我们在这个上下文中对它们不感兴趣。

## 这个想法

记住我们上面的约束:

*   *   现有代码总是做**全文档更新**
    *   在调用者端已经实现了一个变化:**传入的域对象**将会是**完全的或者部分的**，这取决于用例
    *   Solr 文档实例价值化**跨越了许多方法**。创建一个 *SolrInputDocument* 实例，然后传递给几个设置文档状态的方法。
    *   我们需要**部分更新**，但它们**不会是唯一的场景**:在某些情况下，我们仍然有**完全更新**

在 Java 中实现到目前为止描述的部分更新机制要求方法 *addField* 、 *setField* 或 *removeField* 知道它们的执行上下文(部分或完全更新)。因为在完全更新的情况下，添加一个新的作者会很简单

```
doc.addField(“author”, “Ralph Johnson”);  
```

而在**部分更新**时，有必要考虑**第一次**发生添加时的差异:

```
List<String> authors = new ArrayList();
authors.add(“Ralph Johnson”);
doc.addField(“author”,  new HashMap() {{ “add”, authors}};
```

从随后的时间:

```
Map<String, Object> fieldModifier = 
            (Map<String,Object>)doc.getFieldValue(“author);

List<String> authors = (List<String>) fieldModifier.get(“add”);
authors.add(“John Vlissides”);
```

上面的逻辑(可以写得更好)需要为每个添加/设置/删除调用的字段完成！有没有更好的处理方法？是的，当然:

T12

创建 *SolrInputDocument* 的子类:

```
public class PartialSolrInputDocument extends SolrInputDocument {
     static Function<String, List<Object>> LAZY_EMPTY_MUTABLE_LIST = 
                key -> new ArrayList<>();

     @Override
     @SuppressWarnings("unchecked")
     public void addField(String name, Object value) {
         Map<String, List<Object>> fieldModifier =
                 (Map<String, List<Object>>)computeIfAbsent(name, k -> {
                     remove(name);
                     setField(name, newFieldModifier("add"));

                     return getField(name);
                 }).getValue();

        ofNullable(value)
             .ifPresent(v -> 
                      fieldModifier.computeIfAbsent(
                                fieldModifier
                                  .keySet()
                                  .iterator()
                                  .next(),
                                LAZY_EMPTY_MUTABLE_LIST).add(v));
     }

     @Override
     public SolrInputField removeField(String name) {
        setField(name, newFieldModifier("set"));
        return getField(name);
     }

     private Map<String, List<String>> newFieldModifier(String op) {
        return new HashMap<>()
        {{
           put(op, null);
        }};
     }
}
```

这个类的逻辑可以总结如下:

*   *   **setField** :保持原来的语义:调用此方法将替换任何现有的值
    *   **removeField** :部分文档上的 removeField 表示“嘿，我想从索引文档中删除任何现有的值”。这种语义在原子更新中使用带有空值的**“设置”修饰符**来实现
    *   **addField** :这里的逻辑根据调用“removeField”之前是否发生(在给定的字段上)而改变。
        *   如果字段 X 发生了一个 *removeField* ，它与一个**“set”修饰符**和一个**空值**相关联。然后调用“addField ”,添加的值填充与“set”修饰符相关联的列表。换句话说，意思是“Solr，取这个字段定义，用它替换索引文档中的现有值”。
        *   否则(字段 X 没有发生*remove field*):*add field*在**“add”字段修饰符**中收集一组值。换句话说，收集的值**被添加**到索引文档中的现有值。

使用这种方法，问题:**我们应该使用完全更新还是部分更新？**在对象构建时**是否解决。**

让我们看一个例子。下面是一个现有的方法，它使用一个输入 *SolrInputDocument* 实例，并向多值字段“author”添加一个新名称:

```
public void addAuthor(SolrInputDocument doc, String authorName) {
    doc.addField(“author”, authorName)
}   
```

现在，假设创建 *SolrInputDocument* 实例**的方法知道上下文(全部或部分更新**):

```
// In case of full document update
SolrInputDocument doc = new SolrInputDocument();
```

或者

```
// in case of partial document update (i.e. atomic update)
SolrInputDocument doc = new PartialInputDocument();
```

然后，**不管前面的选择**，下面的方法正确执行:

```
addAuthor(doc, “Ralph Johnson”); 
```

根据传递的 *SolrInputDocument* 实例类型，调用适当的 *addField* 方法，结果文档触发**完全或部分**更新。这同样适用于填充文档状态的所有其他方法。

需要强调的是，这些方法的签名没有改变，**多态**根据类型管理正确的实现。

作为旁注，请记住 *SolrInputDocument* (以及*partialrinputdocument*子类)是成为**脆弱类**[【3】](https://web.archive.org/web/20220925175711/https://en.wikipedia.org/wiki/Fragile_base_class)的潜在候选。
这意味着上述内容并不打算作为适合任何可能情况的通用解决方案。

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220925175711/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220925175711/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们也提供这些话题的咨询，[如果你想让你的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220925175711/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这篇关于原子更新的文章吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！****