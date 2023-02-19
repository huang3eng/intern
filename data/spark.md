# Spark

> [大数据处理框架Apache Spark设计与实现](https://book.douban.com/subject/35140409/)、[学习视频](https://www.youtube.com/watch?v=rQzpxnoZSg8&list=RDCMUC6_yO5_oQfwelde0uEKiSZg&start_radio=1&t=7s)

## MapReduce

- 三驾马车：GFS、MapReduce和Bigtable。分别对应分布式文件系统、分布式计算框架和分布式数据存储
- MapReduce来源于古老的函数式编程思想
  - Map：数据一对一映射，主要完成数据转换的操作
  - Reduce：归约或者化简，主要完成聚合的操作

## Hadoop

> HDFS+MapReduce

- Hadoop1.0的缺陷：主节点可靠性差、资源管理和作业调度没有分开、不支持其他异构计算框架

- Hadoop2.0的主要改进是引入了资源管理与调度系统YARN，计算框架变成了YARN的用户，YARN管理了内存和CPU两种资源

- Hadoop生态

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231150546695.png" alt="image-20221231150546695" style="zoom:50%;" />

## 统一资源管理和调度

- 调度器：集中式调度器、双层调度器（中央调度器和框架调度器）、状态共享调度器（类似MVCC乐观并发机制）

- YARN：主从架构设计

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231151128651.png" alt="image-20221231151128651" style="zoom:50%;" />

- Spark on Yarn可以看成是Driver首先在资源管理和调度系统中注册为框架调度器，接受到所需要的资源后，再开始进行作业调度。

## Spark

### 分析场景

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231151752795.png" alt="image-20221231151752795" style="zoom:50%;" />

### 编程语言部署

#### 编程语言

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231152011995.png" alt="image-20221231152011995" style="zoom:50%;" />

#### 部署

- 选择统一资源管理与调度系统：standalone、K8s、YARN、Mesos
- 提交Spark作业
  - 以YARN 为例，需要将 YARN的配置文件复制到Spark 客户端的配置文件夹下，就可以从该节点向大数据平台提交作业。

### Spark架构

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231152618826.png" alt="image-20221231152618826" style="zoom:50%;" />

- Standalone版本，Spark部署的系统架构图

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230101115711619.png" alt="image-20230101115711619" style="zoom:50%;" />

  - Master节点上常驻Master进程。该进程负责管理全部的Worker节 点，如将Spark任务分配给Worker节点，收集Worker节点上任务的运行 信息，监控Worker节点的存活状态等。
  - Worker节点上常驻Worker进程。该进程除了与Master节点通信， 还负责管理Spark任务的执行，如启动Executor来执行具体的Spark任务 ，监控任务运行状态等。
  - Spark application，即Spark应用，指的是1个可运行的Spark程序， 如WordCount.scala，该程序包含main()函数，其数据处理流程一般先 从数据源读取数据，再处理数据，最后输出结果。同时，应用程序也包 含了一些配置参数，如需要占用的CPU个数，Executor内存大小等。
  - Spark Driver，也就是Spark驱动程序，指实际在运行Spark应用中 main()函数的进程，官方解释是“The process running the main() fun ction of the application and creating the SparkContext”
  - Executor，也称为Spark执行器，是Spark计算资源的一个单位。**Executor在物理上是 一个JVM进程**，可以运行多个线程(计算任务)。
  - task，即Spark应用的计算任务。**task以线程方式运行 在Executor进程中**，执行具体的计算任务，如map算子、reduce算子等。 由于Executor可以配置多个CPU，而1个task一般使用1个CPU，因此当E xecutor具有多个CPU时，可以运行多个task。

- [Spark中Task，Partition，RDD、节点数、Executor数、core数目的关系和Application，Driver，Job，Task，Stage理解](https://www.cnblogs.com/AlanWilliamWalker/p/11712035.html)

- 执行步骤

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231153054731.png" alt="image-20221231153054731" style="zoom:50%;" />



### 核心数据结构RDD

#### 基本概念

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231153325926.png" alt="image-20221231153325926" style="zoom:50%;" />



#### 例子

> RDD是分区的集合，本质上是一个集合。RDD_0为PartitionRDD、RDD_1为ShuffledRDD

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231153548911.png" alt="image-20221231153548911" style="zoom:50%;" />

#### 创建RDD

- 将内存中的集合变量转换为RDD
- 从HDFS中读取
- 从外部数据源读取
  - Spark 从MySQL中读取数据返回的RDD类型是JdbcRDD
  - 也可以从类似HBase这种分布式数据库中读取
- PairRDD：PairRDD与其他RDD并无不同，只不过它的数据类型是Tuple2[K，V]，表示键值对。这种数据结构决定了PairRDD可以使用某些基于键的算子，如分组、汇总等

### 如何用算子构建数据管道

#### 不同种类的算子

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231155443279.png" alt="image-20221231155443279" style="zoom:50%;" />

#### 转换算子

- 按照不同依据分类
  - 根据分区与分区之间的映射关系分类：一对一（map）、多对一（union）、多对多（groupByKey）
  - 根据RDD的结构分类：value型RDD、key-value型RDD
  - 根据用途分类：通用类、数学/统计类、集合论与关系类、数据结构类
    - 通用类：map、filter、reduceByKey、groupByKey、flatMap等
    - 数学/统计类：sampleByKey等
    - 集合论与关系类：cogroup（基础）、交集、差集、并集、笛卡儿积等
    - 数据结构类：partitionBy、coalesce、repartition
- 几乎所有的算子都可以用map、reduce、filter这三个算子通过组合进行实现

#### 行动算子

- 行动算子从功能上来说作为一个触发器，会触发提交整个作业并开始执行

- 分类

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231160834569.png" alt="image-20221231160834569" style="zoom:50%;" />

- spark缓存算子也属于行动算子，也就是说会触发整个作业开始计算，例如cache和persist算子。

#### 习题

- 用 Spark算子实现对1TB的数据进行排序？
  - 错误：先分区排序，再汇总排序的操作不试用。因为1TB的数据汇总在一个Executor中会有很大的压力。
  - 正确：首先数据会按照键的区间进行分发，也就是Shuffle。如[0，100000]、[100000，200000)和[200000，300000]，每个分区没有交集。照此规则分发后，分区内再进行排序，就可以在满足性能要求的前提下完成全排序。
  - Spark将上述封装成sortByKey算子，分区器采用RangePartitioner。
- 进阶：如果数据没均匀分布，导致某个分区数据特别多，同样会导致作业失败，如果避免这种数据倾斜的问题。

### 共享变量

#### 广播变量

- 广播变量类似于MapReduce中的DistributeFile，通常来说是一份不大的数据集
- 一旦广播变量在Driver中被创建，整个数据集就会在集群中进行广播能让所有正在运行的计算任务**以只读方式访问**
- 广播变量支持一些简单的数据类型，如整型、集合类型等。也支持很多复杂数据类型，如一些自定义的数据类型
- spark广播机制运作方式
  - Driver将已序列化的数据切分成小块，然后将其存储在自己的块管理器BlockManager中
  - 当Executor开始运行时，每个Executor首先从自己的内部块管理器中试图获取广播变量
  - 如果以前广播过，那么直接使用。如果没有，**Executor就会从Driver 或者其他可用的Executor去拉取数据块**
  - 一旦拿到数据块，就会放到自己的块管理器中，供自己和其他需要拉取的Executor使用。这就很好地防止了Driver单点的性能瓶颈
- 典型场景
  - 有表A和表B，表A（校验码，内容），表B（校验码，规则）。需要根据校验码的不同，对内容才去不同规则的校验。
  - 将表B进行广播，广播到每个Executor的内存中，供处理表A的map函数使用，避免了Shuffle

#### 累加器

- 初始化累加器，行动算子触发计算后，累加器在map函数中被调用，其值会一直增加。
- 常用累加器： LongAccumulator、DoubleAccumulator、CollectionAccumulator[T]和自定义累加器

### Spark Shuffle原理

#### RDD血统

- 在DAG中，最初的RDD被称为基础RDD，后续生成的RDD都是由算子以及依赖关系生成的。也就是说，无论哪个RDD 出现问题，都可以由这种依赖关系重新计算而成。这种依赖关系被称为RDD血统（lineage）。
- 血统表现形式分为窄依赖宽依赖：如果parent RDD的一个或者多个分区 中的数据需要全部流入child RDD的某一个或者多个分区，则是窄依赖 。如果parent RDD分区中的数据需要**一部分流入child RDD的某一个分区**，**另外一部分流入child RDD的另外分区**，则是宽依赖。
- 宽依赖必然会发生Shuffle操作，Shuffle也是划分Stage的依据。
- 适当的调用RDD的checkpoint方法，保存当前计算好的中间结果，依赖链就会大大缩短
- RDD的血统机制就是RDD的容错机制

#### Spark容错

- Driver执行失败是Spark应用最严重的一种情况，标志整个作业彻底执行失败，需要开发人员手动重启Driver
- Executor报错通常是因为Executor所在的机器故障导致，这时Driver 会将执行失败的Task。调度到另一个Executor继续执行，重新执行的Task 会根据RDD的依赖关系继续计算。并将报错的Executor从可用Executor的列表中去掉
- Spark 会对执行失败的Task进行重试，重试3次后若仍然失败会导致整个作业失败。在这个过程中，Task的数据恢复和重新执行都用到了 RDD的血统机制

#### Spark Shuffle

- 有些算子都会引起RDD中的数据进行重分区，新的分区被创建，旧的分区被合并或者被打碎。在重分区的过程中，如果数据发生了跨节点移动，就被称为Shuffle。
- Shuffle机制中最基础的两个问题：数据分区问题和数据聚合问题
  - 第一个问题，分区个数与下游stage的task个数一致
  - 第二个问题，数据聚合的本质是将相同Key的record放 在一起，并进行必要的计算，这个过程可以利用C++/Java语言中的Hash Map实现。

## Spark高级编程

### 如何数据结构化数据 

> 基于RDD+算子的组合更多是基于集合完成数据处理的。对于数据处理逻辑，分析师梗习惯用表的概念而非集合的概念。DataFrame + Spark SQL的组合无论从学习成本，还是从性能开销上来说，都显著优于前者组合。
>

#### DataFrame 

> DataFrame=Dataset[row]

- DataFrame创建数据源：json、csv、外源数据库、RDD通过类型反射创建、流式数据源
- DataFrame read和write
- DataFrame查询
  - RDD算子风格：reduce、groupByKey、map、flatMap（需要传入encoder函数或者通过隐式转换）
  - SQL风格：select、where、groupby、pivot（透视）

#### DataSet

> DatasetAPI提供了类型安全的面向对象编程接口

#### Spark SQL 

- Catalyst优化器带来的性能巨大提升，使Spark SQL成为编写 Spark 作业的最佳方式

- 使用Spark SQL流程

  - 先创建临时视图（从DataFrame、Dataset、Hive元数据库中获取元数据信息）

  - 查询语句，Spark的SQL语法源于Presto (一种支持SQL的大规模并行处理技术，适合 OLAP)

    <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231214432521.png" alt="image-20221231214432521" style="zoom:50%;" />

    - HAVING子句与聚合函数以及 GROUP BY子句配合使用，用来过滤分组统计的结果
    - UNION子句用于将多个查询语句的结果合并为一个结果集

### 如何使用函数和自定义函数

#### 函数

- 转换函数：显示的转换一个值的类型
- 数学函数：log、 factorial
- 字符串函数：split、concat、subString
- 二进制函数：bin、base64
- 日期时间函数：current_date、current_timestamp
- 正则表达式函数、json函数、url函数、聚合函数
- 窗口函数：例如row_number()对窗口中的数据依次赋予行号

#### 窗口函数

- 定义窗口函数

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231215411383.png" alt="image-20221231215411383" style="zoom:50%;" />

- 使用窗口函数

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231215452345.png" alt="image-20221231215452345" style="zoom:50%;" />

#### 用户自定义函数

- DataFrame API支持用户自定义函数，自定义函数有两种：UDF和UDAF。
  - UDF是类似于map操作的行处理，一行输入一行输出
  - UDAF是聚合处理，多行输入，一行输出
- UDAF是用户自定义聚合函数，分为两种：un-type UDAF 和 safe-type UDAF

### 总结

- 在任何情况下，都不推荐使用RDD算子

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231221115822.png" alt="image-20221231221115822" style="zoom:50%;" />

- 在其他情况，应优先考虑 Dataset

  - 因为静态类型的特点会使计算更加迅速，但用户必须使用静态语言才行
  - 如 Java与 Scala，像 Python这种动态语言是没有Dataset API的

## 列式存储

> 针对查询场景的极致优化

### Google Dremel

- Dremel: Interactive Analysis of Web-Scale Datasets
  - 提出列式存储和多级执行树
  - 创新之处在于提出了一种支持嵌套数据的列式存储
- 开源实现
  - 实现了Dremel的嵌套列式存储，如Apache Parquet、Apache ORC
  - 实现了Dremel的查询执行架构，也就是多级执行树，如Apache Impala、Aapche Drill与 Presto
- 多级执行树这种技术与 Spark这种MapReduce类型的计算框架完全不同
  - 它类似于一种大规模并行处理，希望以较低的延迟完成查询，所以并行程度要远远大于Spark，但是每个执行者的性能要远远弱于Spark
  - 如果把 Spark看成是对 CPU核心的抽象，那么多级执行树可以看成是对线程的抽象
  - 基于此，多级执行树+列式存储的组合往往用于OLAP的场景

### 列式存储实现

- Parquet 和 ORC这两种数据格式和Json 一样都是自描述数据格式
  - Parquet 支持几乎Hadoop生态圈的所有项目，与数据处理框架、数据结构以及编程语言无关
  - ORC提供ACID 支持、也提供不同级别的索引，如布隆过滤器、列统计信息(数量、最值等)。和 Parquet 一样，它也是自描述的数据格式
- CarbonData是华为开源的一种列式存储格式，是专门为海量数据分析和处理而生的

### 对比测试

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221231224817666.png" alt="image-20221231224817666" style="zoom:50%;" />

- 列式存储作为列的集合，空间几乎没有多余的浪费
- 如此高的压缩效率也带来了一个优化思路：可以将若干相关的表预先进行连接，连接而成的表可以看成是一张稀疏的宽表，这张宽表对分析来说就非常友好了。但由于采用了列式存储，所以**宽表所占的空间并不是指数上涨而是线性增加**
- 列式存储在数据分析领域非常火，俄罗斯开源列式分析数据库ClickHouse

## Spark性能调优

###  日志收集

- driver日志：适用于比较简单的错误
- executor日志
  - 查看Executor的日志需要先将散落在各个节点(Container)的日志收集汇总成一个文件
  - 阅读这样的日志，最重要的是找到最开始报错的那一句日志。因为一旦作业报错，几乎会造成所有Container报错，但大部分错误日志都对定位原因没有什么帮助
  - 拿到这份日志要做的第一件事是利用时间戳和 ERROR标记定位最初的错误日志，这种方式通常可以直接解决一半以上的报错问题

### 硬件配置和资源管理平台

- 每台节点应预留20%的资源保证操作系统与其他服务稳定运行
- 采取YARN的新特性：基于标签的调度，在某些节点上打上相应的标签

### 参数调优和应用调优

- 提高作业并行度：改变并行程度只有一个方法，就是提高同时运行的Executor的个数。

- 提高shuffle的性能

  - 常用参数spark.shuffle.file.buffer、spark.reducer.maxSizelnFlight、spark.shuffle.compress

- Spark内存管理

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230101122900106.png" alt="image-20230101122900106" style="zoom:50%;" />

  - spark.memory.fraction设置框架内存空间、spark.memory.storageFraction设置数据缓存空间

- 序列化

  - 以时间换空间的一种内存取舍方式，根本原因还是内存比较吃紧
  - 因为反序列化时会造成访问时间过慢，如果想用序列化的方式存储数据，推荐使用 **Kyro** 格式。

- JVM垃圾回收（GC）调优

  - 在Spark中，GC调优的目的是确保只有长生命周期的对象才会保存到老年代中。年轻代有充足的空间来存储短生命周期对象，会有助于避免执行full GC来回收任务
  - 如果任务在完成之前多次触发full GC，则意味着没有足够的内存可用于执行任务
  - 如果minor GC次数过多，但并没有 major GC，可以为Eden 区分配更多的内存来缓解

- 将经常被使用的数据用cache算子进行缓存

- 在进行连接操作时，尝试将小表通过广播变量进行广播

- 在使用filter算子后，通常数据会被打碎成很多个小分区，这会影响后面的执行操作，可以先对后面的数据用coalesce算子进行一次合并（**不是特别懂**）

- 根据场景选用高性能算子

- 数据倾斜问题

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230101124419249.png" alt="image-20230101124419249" style="zoom:50%;" />

  - 将不均匀的数据进行单独处理
  - 将数据打上随机的键值，根据键值进行分发，将数据均匀的分散到多个任务中，在每个任务中，按照真实的键值进行局部聚合，再进行分发。

## Spark两个优化提升项目

### Tungsten

- Tungsten项目产生的原因是由于固态硬盘和万兆交换机的普及和应用，I/O性能的大幅提升使得CPU和内存成了大数据处理中的新瓶颈
- Tungsten优化方式
  - 摆脱JVM的垃圾回收器，自己管理内存
  - 直接生成字节码
  - 加速序列化和反序列化对象
  - Catalyst优化器

### Hydrogen

- Hydrogen项目出现的背景是，目前机器学习框架与深度学习框架开始井喷，而Spark的野心在于一统整个数据科学领域，所以也乐见其成
- Hydrogen优化方式
  - 优化数据交换方式：向量化的数据传输
  - 解决Spark计算模型和机器学习框架的不相容性：带有同步栅的执行模型<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230101125749912.png" alt="image-20230101125749912" style="zoom:50%;" />
  - 让spark3.0在Standalone模式、YARN模式、Kubernetes模式中具有GPU感知能力

## 流处理

### 消息送达保证 

- 在这个过程中，如何保证当前节点一定能收到上游计算节点发送的处理结果是一个非常重要的问题，因为它直接影响了流处理结果的正确性，称其为消息送达保证 (delivery guarantee)问题
- 消息送达保证的三种语义
  - 至少送达一次(at least once)：下游节点一定会收到一次上游节点发过来的消息，但也可能会接收到重复的消
  - 至多送达一次(at most once)，下游节点不一定会收到上游节点发来的消息这意味着，有可能上游节点发送的消息，下游节点丢失了，但上游节点不会重发
  - 恰好送达一次(exact once)，下游节点一定会且只会收到一次上游节点发来的消息


- 假设消费者控制偏移量在日志中的位置
  - 至多一次：读取消息，然后在日志中保存偏移量位置，最后处理消息
  - 至少一次：读取消息，处理消息，最后保存消息的偏移量位置
  - 恰好一次：在一个事务中完成处理消息和保存偏移量的位置
    - 两段式提交
    - 将消费者的输出与消费者的偏移量存储在一个地方
    - 幂等输出 
- 端到端的一致性通常来说涉及到输入-处理-输出的全过程

### SparkStreaming

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230101173026496.png" alt="image-20230101173026496" style="zoom:50%;" />

- RDD是DStream 中某个批次的数据，而DStream 代表了一段时间所产生的RDD。通过这种方式，Spark Streaming 把对连续流的处理，变成了对批序列 DStream的处理
- 转换算子
  - 无状态的转换算子：算子作用于数据流中的每个RDD
  - 有状态转换算子：对DStream中某几个RDD进行操作（基于时间窗口，基于整个时间跨度）

### Dataflow

- Dataflow论文导读：[视频](https://www.bilibili.com/video/BV1oG411x7oG/?spm_id_from=333.788&vd_source=f34c6fbf000c38e6ef25a6a0bdaa8453)，[文档](https://nxwz51a5wp.feishu.cn/docx/doxcnRSOOdA8AlbKJjzkLFlZ3ec)
- Google MillWheel系统：低水位机制（low watermark）
- Dataflow模型的本质
  - 一个思维方式的转变（批处理、流处理 -> 有界、无界）
  - 两个时间域：事件时间、处理时间
  - 三个子模型：窗口模型、触发器模型、增量计算模型
  - 四个拆分维度：
    - 计算什么结果。（**What** results are being computed.）
    - 如何按照事件时间来进行计算。（**Where** in event time they are being computed.）
    - 何时触发计算。（**When** in processing time they are materialized.）
    - 早期的计算结果如何在后期被修正。（**How** earlier results relate to later refinements.）

### Structured Streaming

- [是时候放弃 Spark Streaming 转向 Structured Streaming了](https://zhuanlan.zhihu.com/p/51883927)

- Structured Streaming的抽象与架构

  - 处理模式

    ![img](https://pic2.zhimg.com/v2-8467b7d62beb4c6353e9504b0b616e81_r.jpg)

  - Structured Streaming采用的是Google Dataflow的思想，将数据流抽象成无边界表(unboundedtable) 

    <img src="https://pic4.zhimg.com/80/v2-64b6c2bc2092599a95a85e31cb6566f3_1440w.webp" alt="img" style="zoom:48%;" />

  - 整个流处理过程抽象：Source + StreamExecution + Sink

    > 对应**输入-处理-输出**过程，有能力为用户提供端到端的『恰好一次』消息送达语义

    - 输入：Kafka数据源、HDFS数据源、控制台数据源、Socker数据源、Rate数据源
    - 处理：StreamExecution存在的目的是要将流式数据转化为DataFrame的执行计划并执行，其实最后还是由SQLExecution负责执行，也就是与批处理同样的执行引擎，这样的好处就在于无缝对接了DataFrame与Tungsten 巨大的性能优化，并且统一了流处理和批处理的计算引擎
    - 输出：KafkaSink、FileStreamSink、ForeachSink
      -  输出模式有三种：Complete mode、Append mode (default)、Update mode

## MLlib

### 机器学习

- 典型机器学习工作流

  ![image-20230102105222482](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102105222482.png)

- 机器学习任务的学习类型

  - 监督学习
    - 分类：朴素贝叶斯、SVM、随机森林
    - 回归：线性回归、逻辑回归
  - 非监督学习
    - 聚类：k-means
    - 降维：主成分分析、svd
  - 强化学习

### ML Pipeline

- Spark ML API引入了Pipelines API(管道)，类似于Python 机器学习库Scikit-Learn中的Pipeline，采用了一系列API定义，包含了数据收集、预处理、特征抽取、特征选择、模型拟合、模型验证、模型评估等一系列阶段

- 一个Pipeline可以将多个Transformer和 Estimator组装成一个特定的机器学习工作流

  - DataFrame
    - DataFrame与 Spark SQL 中用到的DataFrame一样,是Spark的基础数据结构可以存储文本、特征向量、训练集以及测试集
    - 除了常见的类型，DataFrame还支持 Spark MLlib 特有的Vector类型
  - Transformer
    - Transformer只是描述了一种映射关系，对应了数据转换的过程，它接收一个DataFrame，生成一个新的DataFrame
    - 在涉及特征转换的过程中经常使用，Transformer用于使训练完成后的模型，将特征数据集(测试集)转换为带有预测结果的数据集的场景
    - Transformer必须实现tranform()方法
  - Estimator
    - 从训练完成好的模型也是一个Transformer，Estimator包含了一个可以让数据集拟合出一个Transformer的算法
    - Estimator必须实现fit()方法

- Pipeline例子文档分类

  ![image-20230102110524614](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102110524614.png)

- Spark MLlib常见算法：特征抽取、转换与选择，分类和回归，聚类，协同过滤，频繁项集挖掘

### 数据预处理

> 在机器学习实践中，数据科学家拿到的数据通常是不尽人意的，例如出现存在大量的缺失值、特征的值是不同的量纲、有一些无关的特征、特征的值需要再次处理等情况，这样的数据无法直接训练，因此我们需要对这些数据进行预处理。

- 数据标准化：Z分数法、最大最小法
- 缺失值处理
  - 如果特征值是连续型，通常用中位数来填充
  - 如果特征值是标签型，通常用众数来补齐
  - 某些情况下，还可以用一个显著区别于已有样本中该特征的值来补齐
- 特征抽取与选择
  - 特征抽取：按照某种映射关系生成原有特征的一个特征子集
    - PCA、word2Vector
  - 特征选择：根据某种规则对原有特征筛选出一个特征子集
    - 卡方检验、信息增益、相关系数
  - 共同点：都试图减少数据集中的特征数目
  - 不同点：具体方法不同，特征抽取主要是通过特征间的关系来操作，特征选择是从原始特征数据集中选择出子集，两者是一种包含关系

### 分类

#### 决策树算法

- 定义：决策树是一种机器学习的方法，它通过一种树形结构对样本进行分类，每个非叶子结点代表一次判断，每个叶子结点代表的是分类结果。

- 构造决策树：特征选择、决策树生成、决策树剪枝
  - 特征选择：信息增益（ID3）、信息增益率（C4.5）、基尼系数（CART）
  - 剪枝：预剪枝、后剪枝

#### 随机森林

> 支持分布式计算最好的也是最常用的方法就是随机森林算法，基础是决策树算法

- 定义：随机森林就是通过集成学习的思想将多棵树集成的一种算法，基本单元是决策树
- 集成学习
  - 通过构建多个弱分类器，并按一定规则组合起来的分类系统，常常比单一分类器具有显著优越的泛化性能
  - 常见集成学习算法：随机森林、AdaBoost、XgBoost、梯度提升树
- 随机森林每一棵树随机抽取训练样本，随机选择特征，没有剪枝过程

### 聚类

- 定义：聚类是把相似的对象通过静态分类的方法分成不同的组别或者更多的子集

- k-means算法：

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102115751820.png" alt="image-20230102115751820" style="zoom:50%;" />

  - 在聚类之前，需要先对特征用PCA进行降维，用Z分数进行归一化
  - 评估聚类结果：簇间距离和簇内距离

### 推荐系统

#### 协同过滤

- 基于用户的协同过滤：计算用户的相似性，在相似用户之间进行推荐，在社交类平台效果较好
- 基于商品的协同过滤：计算商品的相似性，购买商品的时候，推荐其相似的商品，在非社交类平台效果较好
- 基于模型的协同过滤：采用矩阵分解，例如$A=U\Sigma V^{T}$或者$R=V^{T}U$

#### 推荐引擎

- 推荐引擎利用用户登录自选标签和注册信息分类进行响应推荐，从而实现冷启动

#### 模型评估和调优 

- 模型评估：精确率、召回率、准确率、ROC与PRC
- 交叉验证：K折交叉验证法
- 超参调优：网格搜索

## GraphX(todo)

## 从数据到分析

> 将企业中现有的数据转化为知识，帮助企业做出明智的业务经营决策的工具，通过对商业信息进行搜集、管理和分析，旨在使企业的各级决策者获得知识或洞察力
>

![截屏2023-01-02 14.00.38](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/%E6%88%AA%E5%B1%8F2023-01-02%2014.00.38.png)



### 数据仓库

> 数据仓库是一个面向主题的、集成的、时变的、非易失的数据集合，支持管理者的决策过程。
>

- 业务数据库和数据仓库的区别
  - 操作数据库系统的主要任务是执行联机事务和查询处理（OLTP）
  - 数据仓库系统在数据分析和决策方面为用户提供服务（OLAP）
- OLTP和OLAP的区别
  - 用户和系统的面向性：OLTP是面向客户的，OLAP是面向市场的
  - 数据内容：OLTP系统管理的是当前数据，OLAP系统管理的是大量的历史数据
  - 视图：OLTP系统主要关注一个企业或部门内部的当前数据，OLAP系统还要处理来自不同组织的信息，以及由多个数据库集成的信息
  - 访问模式： OLTP系统主要由短的原子事务组成，OLAP系统的访问大部分是只读操作
- 数据集市：数据集市往往服务于一组特定群体的分析需求(如会计部分或者信贷部门)，有些数据集市可以独立于数据仓库存在，由业务数据库的历史应用创建。数据集市的用户往往是直接和业务相关的分析应用

### 如何获取数据源

- 数据来源
  - 关系数据库：JDBC数据库
  - NoSQL数据库：MongoDB
- 一般从数据库或者MongoDB导出后，还需要将数据上传到hdfs中
- 从NoSQL数据库的数据导出的过程中，既有命令行，也有Spark代码

### 如何构建数据立方体

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102140918717.png" alt="image-20230102140918717" style="zoom:50%;" />

- 数据从业务数据库中导出到大数据平台，将导出后的数据集合视为数据仓库
- 构建数据立方体，在生成数据立方体的同时，也就成功构建了数据集市
  - 清楚数据立方体的维度、度量
  - 最后保存到文件系统时，采用ORC格式
- 利用调度平台整合脚本，完成整个过程的调度与定时执行
  - Airflow：用 Python实现任务管理、调度、监控工作流的平台

### 多维分析和报表

#### Presto

> 完成我们对数据立方体进行多维分析的需求

- 用于大数据的高性能分布式SQL查询引擎，其架构允许用户查询各种数据源
- Presto可以直接操作 HDFS上的数据

#### Superset

> 将 Presto返回的结果进行可视化呈现

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102141926203.png" alt="image-20230102141926203" style="zoom:50%;" />

### 数据更新和实时性

- 全量数据

  - 存在于数据仓库中的数据
  - 通过转换任务脚本转换而成只在系统上线时执行一次
  - 之后以t为周期对更新数据进行处理

- 增量数据：

  - 以日志的形式进行推送无法直接进行分析
  - 全量数据可以直接进行分析却无法捕捉到最新变化
  - 将增量数据与全量数据进行合并

- 数据更新

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102142412218.png" alt="image-20230102142412218" style="zoom:50%;" />



### Lambda和Kappa架构

- 流程对比

  <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102142833191.png" alt="image-20230102142833191" style="zoom:50%;" />

- 其他维度对比

  ![image-20230102143050277](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102143050277.png)

## 大数据系统的统一

### 统一的编程模型

> 目前主流的大数据处理引擎均选用Dataflow模型作为自己的设计蓝本

- 计算什么(What)
- 根据事件时间，哪些数据会参与计算(Where)
- 什么时候触发计算(When)
- 早期的计算结果如何被修正 (How)

### 统一的编程接口

> Beam是一套统一的编程模型，在任何引擎（apex、flink、spark、google dataflow）上运行批处理和流处理作业
>

<img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102143507161.png" alt="image-20230102143507161" style="zoom:50%;" />

### 统一的架构

> 存储层、资源管理与调度层、计算层

- 计算层：beam是对计算框架的统一（编程接口的编程接口）

- 资源管理和调度层：k8s是对任意计算类型资源的统一（资源管理与调度系统的资源管理与调度系统）

  - 在资源层面YARN没有做到统一，只能调度大数据计算作业，不能调度Web服务和数据库服务之类的计算需求

- 存储层：Allxuio是对任意计算类型资源的统一（文件系统的文件系统）

  - Alluxio是一个提供内存级别读写速度的虚拟分布式存储

    <img src="https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230102144251253.png" alt="image-20230102144251253" style="zoom:50%;" />
