# 音乐信息检索:音程表

> 原文：<https://web.archive.org/web/sease.io/2020/02/music-information-retrieval-intervals-table.html>

在这篇文章中，我们描述了什么是**区间表**以及如何使用**行为驱动开发** (BDD)方法来构建它。

**区间表**已于 2019 年 10 月在我们的[伦敦信息检索会议](https://web.archive.org/web/20220925182009/https://www.meetup.com/it-IT/London-Information-Retrieval-Meetup-Group)中推出(此处[为](https://web.archive.org/web/20220925182009/https://www.slideshare.net/SeaseLtd/musical-information-retrieval-take-2-interval-hashing-based-ranking)幻灯片):它是我们正在研究的**翻唱歌曲检测**方法中的核心组件。实际上，是**“音程”**代表了这种方法中的一个核心概念，因为它允许根据**“相对性”**来思考，而不用关心**绝对频率**或音高。

## 什么是音程？

音乐理论将音程定义为两个声音**之间的**音高差**。**

如果一个音程表示两个**连续**发声音调之间的距离，它可以是**水平**(或**线性**或**旋律**)，如果它属于**同时**发声音调，如在一个和弦中，它可以是**垂直**或**和声**。

![Horizontal and Vertical Intervals](img/a5eaad477cc3a8f8b19c92697f7988a3.png)

在**西洋音乐**中，音程代表一个**全音阶**的**两个音符**之间的**距离**。这些音程中最小的是一个**半音**。

**全音阶**是一种**音阶**具有

*   **每八度七个**音高
*   **五个**双半音(又名**音**)音程
*   **两个**半音音程由**两个**三个音程隔开

c 大调音阶是全音阶的完美例子(1/2 =半音，1 =音调):

![The Diatonic Scale](img/4c78f8f7dc3c45f6b3735cdadbfe7247.png)

在上面的范围内，我们可以说

*   C 和 D 之间的距离是 1 个音(或 2 个半音)
*   D 和 G 之间的距离是 2 个半音(或 5 个半音)
*   A 和 C 之间的距离是 1 个半音(或 3 个半音)

在**西方音乐**中，有哪些音阶

*   **每个**八度**十二个**音高
*   **十二个**半音音程

叫做**半音阶**。它列出了所有可能的音符(同样，在西方音乐中):

![The Chromatic Scale](img/4817d3d9486435e1493f845655074fb3.png)

## 间隔表

**音程表**是一个**矩阵**，它定义了**音阶**中每个音符之间的**可能音程**:

| **C** | **C#** | **D** | **D#** | **E** | **F** | **F#** | **G** | **G#** | **答** | **A#** | **B** |
| **C** | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
| **C#** | 11 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| **D** | 10 | 11 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
| **D#** | 9 | 10 | 11 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
| **E** | 8 | 9 | 10 | 11 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| **F** | 7 | 8 | 9 | 10 | 11 | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| **F#** | 6 | 7 | 8 | 9 | 10 | 11 | 0 | 1 | 2 | 3 | 4 | 5 |
| **G** | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 0 | 1 | 2 | 3 | 4 |
| **G#** | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 0 | 1 | 2 | 3 |
| **答** | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 0 | 1 | 2 |
| **A#** | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 0 | 1 |
| **B** | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 0 |

虽然在**乐理**中，每个音程都用一个名称来表示(例如 0 =同音)，但表格使用半音**的数量**来表示距离。

使用**区间表**我们可以说:

*   A 和 D#之间的距离是 6 个半音
*   B and B 之间的距离是 0 个半音(同音)

如果我们使用基于 **0 的位置**来表示表中的音符(例如，C = 0，C# = 1，D = 2)，音程表可以被视为**一个函数**，该函数**将两个音符** (x，y)映射到**一个数字**，该数字表示半音中的**距离。**

在**斯卡拉**

```
 def distanceBetween(x: Int, y: Int): Int = 
    if (y >= x) end - x else (11 - (x - 1)) + y
```

在 **Clojure**

```
 (defn distance-between [x y]
    (if (>= y x)
        (-y x)
        (+ (- 11 (- x 1)) y)))
```

其用法的一个例子:

```
> val table = new IntervalsTable()

> // 4 = E, 9 = A
> table.distanceBetween(4, 9)
res0: Int = 5

> table.distanceBetween(9, 4)
res0: Int = 7
```

*distanceBetween* 函数计算两个音符之间的半音间隔。从那里开始，有可能在其上创建一些高级结构。

例如， *distanceBetween* 的**重载版本**可以计算两个向量之间的间隔**。这种情况下的**结果**是**另一个向量**，其中每个元素代表源向量的第**个元素**之间的距离:**

```
scala> val table = new IntervalsTable()
scala> table: io.sease.ra.IntervalsTable@16c5b50a

// Vector #1 = (C, E, G) 
scala> val v1 = List(0, 4, 7)
v1: List[Int] = List(0, 4, 7)

// Vector #2 = (E, G, B) 
scala> val v2 = List(4,7,11)
v2: List[Int] = List(4, 7, 11)

scala> table.distanceBetween(v1, v2)
res0: Seq[Int] = List(4, 3, 4)
```

继续向前，在更高层次上，IntervalsTable 应提供计算矩阵中**相邻向量之间距离的功能:**

```
Input Matrix (3 x 4):

0  4  3  1
4  7  9  5
7  11 10 10

    =

Distance between n1 and n2

0 - 4   =  4 
4 - 7   =  7 
7 - 11  =  11  

Distance between n2 and n3
4  - 3  =  11
7  - 9  =  2
11 - 10 =  11

Distance between n3 and n4
3  - 1  =  10
9  - 5  =  8
10 - 10 =  0 

Output matrix (3 x 3):

4  11  10
7  2   8
11 11  0
```

## 实施

与其从 **IntervalsTable** 类开始，不如让我们先定义我们想要实现的行为:

**区间表**

*   当**开始**和**结束**相同时，应返回 **0**
*   当**开始**和**结束**和**注释**为**后续**时，应返回 **1**
*   如果音程为**一个音**(例如 2 个半音)，则应返回 **2**
*   当音符与前一个之间的间隔为**时，应返回 **11****
*   在**下限**小于 0**的情况下，应抛出异常**
*   在**下限**大于 11 的情况下，应抛出异常
*   如果**上限**小于 0**则抛出异常**
*   在**上限**大于 11 的情况下应该抛出异常

Scala 允许在测试方法中直接翻译这些断言(抱歉是水平滚动条，你可以在这里找到相同的代码):

```
class IntervalsTableSpecs extends FlatSpec {

  "The Intervals Table" should "return 0 (min value) when start and end note are the same" in {
    val pipeline = new IntervalsTable()
    0 to 11 by 1 foreach(idx => assert(pipeline.distanceBetween(idx, idx) == 0))
  }

  it should "return 1 when start and end are subsequent" in {
    val pipeline = new IntervalsTable()

    0 to 11 by 1 foreach(idx => assert(pipeline.distanceBetween(idx, if (idx == 11) 0 else idx + 1) == 1))
    11 to 0 by -1 foreach(idx => assert(pipeline.distanceBetween(idx, if (idx == 11) 0 else idx + 1) == 1))
  }

  it should "return 2 when the interval is one tone" in {
    val pipeline = new IntervalsTable()

    0 to 10 by 2 foreach(idx => assert(pipeline.distanceBetween(idx, if (idx >= 10) (10 - idx) else idx + 2) == 2))
    11 to 0 by -2 foreach(idx => assert(pipeline.distanceBetween(idx, if (idx >= 10) (11 - idx) + 1 else idx + 2) == 2))
  }

  it should "return 11 (max value) when the interval is between a note and the preceding" in {
    val pipeline = new IntervalsTable()
    0 to 10 by 2 foreach(idx => assert(pipeline.distanceBetween(idx, if (idx == 0) 11 else idx - 1) == 11))
  }

  it should "throw an exception in case the lower bound is lesser than 0" in {
    val pipeline = new IntervalsTable()

    assertThrows[IllegalArgumentException] {
      pipeline.distanceBetween(-1, 8);
    }
  }

  it should "throw an exception in case the lower bound is greater than 11" in {
    val pipeline = new IntervalsTable()

    assertThrows[IllegalArgumentException] {
      pipeline.distanceBetween(12, 8);
    }
  }

  it should "throw an exception in case the higher bound is lesser than 0" in {
    val pipeline = new IntervalsTable()

    assertThrows[IllegalArgumentException] {
      pipeline.distanceBetween(8, -1);
    }
  }

  it should "throw an exception in case the higher bound is greater than 11" in {
    val pipeline = new IntervalsTable()

    assertThrows[IllegalArgumentException] {
      pipeline.distanceBetween(8, 12);
    }
  }
}
```

最后，这里是 **IntervalsTable** 类:

```
class IntervalsTable {

  /**
    * Returns the distance (in semitones) between 
    * the given start and end note.
    *
    * @param start the lower bound note.
    * @param end the higher bound note.
    * @return the distance between the given start and end note.
    */
  def distanceBetween(start: Int, end: Int): Int = {
    require(start >= 0 && start < 12)
    require(end >= 0 && end < 12)
    if (end >= start) end - start else (11 - (start - 1)) + end
  }

  /**
    * Given two equal-sized vectors t1 and t2, 
    * returns a vector consisting of the distance 
    * between notes at the same index.
    * In the following example t1 is a CMaj chord, t2 is Em:
    *
    * t1 t2        res
    *
    * 0  4   ==>    4
    * 4  7   ==>    3
    * 7  11  ==>    4
    *
    * @param t1 note vector at time t1
    * @param t2 note vector at time t2
    * @return a distance vector between t1 and t2
    */
  def distanceBetween(t1: Seq[Int], t2: Seq[Int]): Seq[Int] = {
    assert(t1.length == t2.length)
    (t1, t2).zipped.map(distanceBetween)
  }
}
```

还有什么？让我们**执行测试**并确保桌子符合预期行为:

```
> sbt test

...
[info] Done compiling.
[info] Formatting 1 Scala source in shared_module:test ...
[info] Reformatted 1 Scala source in shared_module:test

...

[info] Done compiling

...

[info] IntervalsTableSpecs:
[info] The Intervals Table
[info] - should return 0 (min value) when start and end note are the same
[info] - should return 1 when start and end are subsequent
[info] - should return 2 when the interval is one tone
[info] - should return 11 when the interval is between a note and the preceding
[info] - should throw an exception in case the lower bound is lesser than 0
[info] - should throw an exception in case the lower bound is greater than 11
[info] - should throw an exception in case the higher bound is lesser than 0
[info] - should throw an exception in case the higher bound is greater than 11

...

[info] Total number of tests run: 8
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 8, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
```

同样的执行，在 IntelliJ:

![This image has an empty alt attribute; its file name is test-execution.png](img/d9de76cd8317cfff4961da366a7935be.png)

*未完待续下集……*

// our service

## 不要脸的塞给我们培训和服务！

我提到过我们做 [Apache Solr 初学者](https://web.archive.org/web/20220925182009/https://sease.io/training/apache-solr-training/apache-solr-beginner-training)和 [Elasticsearch 初学者](https://web.archive.org/web/20220925182009/https://sease.io/training/elasticsearch-trainings/elasticsearch-beginner-training)培训吗？
我们还提供关于这些主题的咨询，[如果您想让您的搜索引擎更上一层楼，请联系](https://web.archive.org/web/20220925182009/https://sease.io/contacts)！

// STAY ALWAYS UP TO DATE

## 订阅我们的时事通讯

你喜欢这个关于音乐信息检索中音程表的帖子吗？不要忘记订阅我们的时事通讯，以便随时了解信息检索世界的最新动态！