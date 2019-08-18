# Spark Streaming

 - SparkStreaming是Spark CoreAPI的一种扩展，它可以用于进行大规模、高吞吐量、容错的实时数据流的处理。

 - 支持从很多数据源中读取数据，比如Kafka、Flume、Twitter、ZeroMQ、Kinesis或者TCP Socket。

 - 可以使用类似高阶函数的复杂算法来进行数据处理，比如map、reduce、join、window。处理后的数据可以被保存到文件系统、数据库、Dashboard等存储中。

   

## DStream以及基本工作原理

 - 接收实时输入数据流，然后将数据拆分成多个batch，比如每收集1秒的数据封装为一个batch，然后将每个batch交给Spark的计算引擎进行处理，最后会生产出一个结果数据流，其中的数据，也是由一个一个的batch所组成的。
- Spark Streaming提供了一种高级的抽象，叫做DStream（Discretized Stream "离散流"），代表一个持续不断的数据流
  - DStream可以通过输入数据源来创建比如Kafka、也可以通过对其他DStream应用高阶函数来创建如map、reduce、join、window。
  - DStream的内部，其实是一系列不断产生的RDD。RDD是Spark Core的核心抽象——不可变的、分布式的数据集。DStream中每个RDD都包含了一个时间段内的数据。
- 对DStream应用的算子，比如map其实在底层会被翻译成对DStream中每个RDD的操作。比如对一个Dstream执行一个map操作，会产生一个新的DStream。但是，在底层，其实其原理为：对输入DStream中每个时间段的RDD，都应用一边map操作，然后生成新的RDD，即作为新的DStream中的个时间端的一个RDD。底层的RDD的transformation操作，其实，还是由Spark Core的计算引擎来实现的。Spark Streaming对Spark Core进行了一层封装，隐藏了细节，然后对开发人员提供了方便易用的高层次的API。



## 与Storm的对比分析

| 对比点       | Storm                  | Spark Streaming                                         |
| ------------ | ---------------------- | ------------------------------------------------------- |
| 实时计算模型 | 纯实时，来一条处理一条 | 准实时，对一个时间段内的数据收集起来，作为一个RDD再处理 |
|实时计算延迟度 | 毫秒级 |秒级|
|吞吐量 | 低 |高|
|事务机制 | 支持完善 |支持，但不完善|
|健壮性/容错性 | Zoookeeper，Acker，非常强 |CheckPoint，WAL，一般|
|动态调整并行度 | 支持 |不支持|

- SparkStreaming 与Sorm的优劣分析
  - 两个框架再实时计算领域中都很优秀，只是擅长的领域不同
  - Spark Streaming仅仅在吞吐量上要比Storm要优秀。但不是所有场景下都注重吞吐量。
  - Storm在实时延迟度上，比Spark Streaming好多了，前者是纯实时，后者是准实时。而且Storm的事务机制、健壮性/容错性、动态调整并行度等特性，都要比Spark Streaming更加优秀。
  - 但是Spark Streaming，位于Spark生态技术栈中，因此SparkStreaming可以和Spark Core、Spark SQL无缝整合，我们可以实时处理出来的中间数据，立即在程序中无缝进行延迟批操作、交互式查询等操作。这个特点大大增强了Streaming的优势和功能。

### Spark Streaming 与Storm的应用场景

 - 对于Storm来说：
    - 建议在需要纯实时，不能忍受1秒以上延迟的场景使用，比如实时金融系统、要求纯实时进行金融交易和分析。
    - 对于实时计算的功能中，要求可靠的事务机制和可靠性机制，即数据的完全精准，一条也不能多不能少，考虑使用storm
    - 如果还需要针对高低峰时间段动态调整计算程序的并行度，以最大限度利用集群资源（小公司，集群资源紧张）。
    - 对一个大数据应用系统，它就是纯粹的实时计算，不需要在中间执行SQL交互式查询、复杂的transformation算子等。
- 对于Spark Streaming来说
  - 如果对于上述适合Storm的三点（纯实时，事务机制和可靠性、动态调整并行度），一条都不满足的实时场景（不要求纯实时，不要求强大可靠的事务机制、不要求动态调整并行度），可以考虑使用Spark Streaming。
  - 考虑使用Spark Streaming的最主要一个因素，应该对整个项目进行宏观的考虑，即如果一个项目除了实时计算之外，还包括了离线批处理、交互式查询、等业务功能，而且实时计算中可能会牵扯到高延迟批处理、交互式查询等功能，那么就应该首选Spark生态，用Spark Core开发离线批处理，用Spark SQL开发交互式查询，用Spark Streaming开发实时计算，三者可以无缝整合，给系统提供非常高的可扩展性。

## StreamingContext详解

 - 有两种创建StreamingContext的方式
    - 第一种
       - ` val conf = new SparkConf().setAppName(appName).setMaster("local[2]")`
       - `val ssc = new StreamingContext(conf,Seconds(1))`
   - 第二种
     - SptreamingContext还可以使用已有的SparkContext来创建
     - ` val sc = new SparkContext(conf)`
     - `val ssc = new StreamingContext(sc,Seconds(1))`
   - appName,是用来在Spark UI上显示的应用名称。master，是一个Spark、Mesos或者Yarn集群的URL或者是local[*]
   - batch interval 可以格局你的应用程序的延迟要求以及可用的集群资源情况来设置。
- 一个StreamingConext定义以后，必须做以下几件事情：
  - 1通过创建DStream来创建数据源
  - 2通过对DStream定义transformatition和output算子操作，来定义实时计算逻辑
  - 3 调用StreamingContext的start()方法，来开始实时处理数据。
  - 4通过StreamingContext的awaitTermination（）方法，来等待应用程序的终止。
  - 5 也可以通过调用StreamingContext的stop（）方法，来停止应用程序。
- 需要注意的要点；
  - 1只要一个StreamingContext启动后，就不能再往里面添加计算逻辑，即执行start()方法后，不能再增加任何任何算子了。
  - 2 一个StreamingContext停止之后，是肯定不能够重启的。调用stop（）方法后，不能再调用start()'
  - 3一个JVM同时只能有一个StreamingContext启动。在你的应用程序种，不能创建两个StreamingContext。
  - 4 调用stop()方法时，会同时停止内部的SparkContext，如果不希望如此，还希望后面继续使用SparkContext，可以使用stop(false)。
  - 一个SparkContext可以创建多个StreamingContext，只要上一个先用stop(false)停止，再创建下一个即可。



## DStream和Receiver详解

 - DStream 代表了来自数据源的输入数据流。除了文件数据流之外，所有的输入DStream都会绑定一个Receiver对象，该对象是一个关键的组件，用来从数据源接收数据，并将其存储在Spark的内存中，以供后续处理。
- Spark Streamig提供了两种内置的数据源支持：
  - 1基础数据源：StreamingContext API中直接提供了对这些数据源的支持，比如文件、socket。
  - 2高级数据源：诸如Kafka、Flume、Kinesis、Twitter等数据源，通过第三方工具类提供支持。这些数据源的使用，需要引用其依赖。
  - 3自定义数据源：我们可以自己定义数据源，来决定如何接受和存储数据。
- 如果想要在实时计算应用中并行接收多条数据流，可以创建多个输入DStream。这样就会创建多个Receiver，从而并行地接收多个数据流。注意，一个Spark Streaming Application的Executor，是一个长时间运行的任务，因此它会独占分配给Spark Atreaming Application的cpu core。从而只要Spark Streaming运行起来以后，它使用的cpu core就没法给其他应用使用了。
- 使用本地模式运行程序时，绝对不能用local或local[1],因为那样的话，只会给执行输入Dstream的executor分配一个线程。而Spark Streaming底层的原理是，至少要有两条线程，一条线程用来分配Receiver接收数据，一条线程用来处理接收到的数据，因此必须使用local[n]，n>=2的模式。



## DStream Kafka数据源 Receiver方式

 - 这种方式使用Receiver来获取数据，Receiver是使用Kafka的高层次Consumer API来实现的。Receiver从Kafka中获取的数据都是存储在Spark Executor的内存中的，然后Spark Straming启动的job会去处理那些数据。
 - 默认配置下，这种方式可能会因为底层的失败而丢失数据。如果要启用高可靠机制，让数据零丢失，就必须启用Spark Streaming的预写日志机制（WAL)。让该机制会同步地接收到的Kafka数据写入分布式文件系统如（HDFS)上的预写日志中。所以，即使底层节点出现了失败，也可以使用日志中的数据进行回复。
 - 注意事项
    - 1此时Kafka中的topic的partition与Spark中的RDD的partition是没有关系的。所以在KafkaUtils.createStream()中，提高partition的数量，只会增加一个Receiver中读取partition的线程的数量。不会增加Spark处理数据的并行度。
    - 2**可以创建多个Kafka输入DStream，使用相同的consumer group和topic，来通过多个receiver并行接收数据。**
    - 3 如果基于容错的文件系统，比如HDFS，启用了预写日志机制，接收到的数据斗湖被赋值到预写日志中。因此，在KafkaUtils。createStream()中，设置的持久化级别是StorageLevel.MEMORY_AND_DISJ_SER.

## DStream Kafka数据源 Direct方式

 - Spark1.3引入了不基于Receiver的直接方式，从而能够确保更加健壮的机制。替代掉使用Receiver来接收数据后，这种方式会周期性地查询Kafka，来获得每个topic+partition的最新offset，从而定义每个batch的offset的范围。当处理数据的job启动时，就会使用Kafka的简单consumer api来获取Kafka指定offset范围的数据。
 - Driect优点
    - 1简化并行读取：如果要读取多个partition，不需要创建多个输入DStream然后对他们进行union操作。Spark会创建跟Kafka partition一样多的RDD partition，并且会并行从Kafka中读取数据。所以在Kafka partition和RDD partition之间，有一个一对一的映射关系
    - 2高性能：如果要保证零数据丢失，在基于receiver的方式中，需要开启WAL机制。这种方式其实效率低下，因为数据实际被复制了两份，Kafka自身就有高可靠的机制，只要Kafka中做了数据的复制，那么就可以通过Kafka的副本进行恢复。
    - 3 一次且仅一次的事务机制：
       - 基于receiver的方式，是使用Kafka的高阶API来在ZooKeeper中保存消费过的offset的。这是消费Kafka数据的传统方式。这种方式配合着WAL机制可以保证数据零丢失的高可靠性，但是无法保证数据被处理一次且仅一次，可能会处理两次。因为Spark和ZooKepper之间是不同步的。
       - 基于Direct的方式，是使用Kafka的简单api，Spark Streaming自己就负责追踪消费offset，并且保存在checkpoint中。Spark自己一定是同步的，因此可以保证数据是消费一次且仅消费一次。



### DStream的transform操作概览

| Transformation   | Meaning                                                      |
| ---------------- | ------------------------------------------------------------ |
| map              | 对传入的每个元素，返回一个                                   |
| flatMap          | 对传入的每个元素，返回一个或多个元素                         |
| filter           | 对传入的元素返回true或false，返回false的元素被过滤掉         |
| union            | 将两个DStream进行合并                                        |
| count            | 返回元素的个数                                               |
| reduce           | 对所有values进行聚合                                         |
| countByValue     | 对元素按照值进行分组，对每个组进行技术，最后返回<K,V>的格式  |
| reduceBYKey      | 对key对应的values进行聚合                                    |
| cogroup          | 对两个Dstream进行连接操作，一个key连接起来的两个RDD的数据，都会以Iterable<V>的形式，出现在一个Tuple中 |
| join             | 对两个Dstream进行join操作，每个连接起来的pair，作为DStream的RDD的一个元素 |
| transform        | 对数据进行转换操作【当DStream的API不够用的时候使用这个函数把DStream转成RDD来操作，最后再返回一个新的RDD】 |
| updateStateByKey | 为每个key维护一份state，并进行更新                           |
| window           | 对滑动窗口数据执行操作（实时计算中最有特点的一种操作）       |



## 缓存与持久化机制+checkpoint机制

## 容错机制和事务语义详解

## 性能调优

