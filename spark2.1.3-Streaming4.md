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
  - 2 一个StreamingContext停止之后，是肯定不能够重启的。调用stop（）方法后，不能再调用start()
  - 3一个JVM同时只能有一个StreamingContext启动。在你的应用程序中，不能创建两个StreamingContext。
  - 4 调用stop()方法时，会同时停止内部的SparkContext，如果不希望如此，还希望后面继续使用SparkContext，可以使用stop(false)。
  - 一个SparkContext可以创建多个StreamingContext，只要上一个先用stop(false)停止，再创建下一个即可。



## DStream和Receiver详解

 - DStream 代表了来自数据源的输入数据流。除了文件数据流之外，所有的输入DStream都会绑定一个Receiver对象，该对象是一个关键的组件，用来从数据源接收数据，并将其存储在Spark的内存中，以供后续处理。
- Spark Streamig提供了两种内置的数据源支持：
  - 1基础数据源：StreamingContext API中直接提供了对这些数据源的支持，比如文件、socket。
  - 2高级数据源：诸如Kafka、Flume、Kinesis、Twitter等数据源，通过第三方工具类提供支持。这些数据源的使用，需要引用其依赖。
  - 3自定义数据源：我们可以自己定义数据源，来决定如何接受和存储数据。
- 如果想要在实时计算应用中并行接收多条数据流，可以创建多个输入DStream。这样就会创建多个Receiver，从而并行地接收多个数据流。注意，一个Spark Streaming Application的Executor，是一个长时间运行的任务，因此它会独占分配给Spark Streaming Application的cpu core。从而只要Spark Streaming运行起来以后，它使用的cpu core就没法给其他应用使用了。
- 使用本地模式运行程序时，绝对不能用local或local[1],因为那样的话，只会给执行输入Dstream的executor分配一个线程。而Spark Streaming底层的原理是，至少要有两条线程，一条线程用来分配Receiver接收数据，一条线程用来处理接收到的数据，因此必须使用local[n]，n>=2的模式。



## DStream Kafka数据源 Receiver方式

 - 这种方式使用Receiver来获取数据，Receiver是使用Kafka的高层次Consumer API来实现的。Receiver从Kafka中获取的数据都是存储在Spark Executor的内存中的，然后Spark Straming启动的job会去处理那些数据。
 - 默认配置下，这种方式可能会因为底层的失败而丢失数据。如果要启用高可靠机制，让数据零丢失，就必须启用Spark Streaming的预写日志机制（WAL)。让该机制会同步地接收到的Kafka数据写入分布式文件系统如（HDFS)上的预写日志中。所以，即使底层节点出现了失败，也可以使用日志中的数据进行回复。
 - 注意事项
    - 1此时Kafka中的topic的partition与Spark中的RDD的partition是没有关系的。所以在KafkaUtils.createStream()中，提高partition的数量，只会增加一个Receiver中读取partition的线程的数量。不会增加Spark处理数据的并行度。
    - 2**可以创建多个Kafka输入DStream，使用相同的consumer group和topic，来通过多个receiver并行接收数据。**
    - 3 如果基于容错的文件系统，比如HDFS，启用了预写日志机制，接收到的数据会被复制到预写日志中。因此，在KafkaUtils。createStream()中，设置的持久化级别是StorageLevel.MEMORY_AND_DISJ_SER.

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

### DStream的transformation操作

​	

| Output                          | Meaning                                                      |
| ------------------------------- | ------------------------------------------------------------ |
| print                           | 打印每个batch中前10个元素，主要用于测试，或者是不需要执行什么output操作时，用于简单触发一些job。 |
| saveAsTextFile(prefix,[suffix]) | 将每个batch的数据保存到文件中。每个batch的文件命名格式为：prefix-TIME_IN_MS[.suffix] |
| saveAsObjectFile                | 同上，但是将每个batch的数据以序列化对象的方式，保存到SequenceFile中。 |
| saveHadoopFile                  | 同上，将数据保存到Hadoop文件中。                             |
| **foreachRDD**                  | 最常用的output操作，遍历DStream中的每个产生的RDD，进行处理。可以将每个RDD中的数据写入外部存储，比如文件、数据库、缓存等。通常在其中是针对RDD执行action操作的，比如foreach。<br />【和transform实现的功能类似，都可以操作RDD，区别是transform有返回值，foreachRDD没有返回值】 |

### output操作

 - DStream中所有的计算，都是由output操作触发的，比如print().如果没有任何output操作，那么压根就不会执行定义的计算逻辑。
 - 使用foreachRDD outprint操作，也必须在里面对RDD执行action操作，才能触发对每一个batch的计算逻辑。否则，光影foreachRDD output操作，在里面没有对RDD执行action操作，也不会触发任何逻辑。

### ForeachRDD 详解

 - 通常在foreachRDD中，就会创建一个Conection，比如JDBC Connection，然后通过Connection将数据写入外部存储。
    - 误区1：在RDD的foreach外部，创建Connection
       - 它会导致Connection对象被序列化后传输到每个Task中。而这种Connection对象，实际上一般是不支持序列化的，也就无法被传输。
   - 误区2：在RDD的foreach操作内部，创建Connection
     - 这种方式是可以的，但是效率低下。因为它会导致对于RDD的每一条数据，都创建一个Connection对象，通常来说，Connection的创建是很消耗性能的。
   - 合理方式1：使用RDD的foreachPartition操作，并且在该操作内部，创建Connection对象，这样就相当于是，为RDD的每个partition创建一个Connection对象，节省资源的多了。
   - 合理方式二：自己手动封装一个静态连接池，使用RDD的foreachPartition操作，并且在该操作内部，从静态连接池中，通过静态方法，获取到一个连接，使用之后再还回去。这样的话，甚至在多个RDD的partition之间，也可以复用连接了。而且可以让连接池采取懒创建的策略，并且空闲一段时间后，将其释放掉。

### window滑动窗口操作

​		

| Transform             | 意义                                       |
| --------------------- | ------------------------------------------ |
| window                | 对每个滑动窗口的数据执行自定义的计算。     |
| countByWindow         | 对每个滑动窗口的数据执行count操作。        |
| reduceByWindow        | 对每个滑动窗口的数据执行reduce操作。       |
| reduceByKeyAndWindow  | 对每个滑动窗口的数据执行reduceByKey操作。  |
| countByValueAndWindow | 对每个滑动窗口的数据执行countByValue操作。 |



## 缓存与持久化机制+checkpoint机制

 - Spark Streaming数据流中的数据持久化到内存中
    - 对DStream调用persist()方法，就可以让Spark Streaming自动将该数据流中的所有产生的RDD都持久化到内存中。
    - 如果要对一个Dstream多次执行操作，那么对DStream持久化是非常有用的。因为多次操作可以共享使用内存中的一份缓存数据。
- 对于基于窗口的操作，比如reduceByWindow、reduceByKeyAndWindow,以及基于状态的操作，比如updateStateByKey，默认就隐式开启了持久化机制。即Spark Streaming默认就会将基于窗口创建的DStrean中的数据缓存到内存中，不需要我们手动调用persist()方法。
- 对于通过网络接收数据的输入流，比如socket、Kafka等，默认的持久化级别，是将数据复制一份，以便容错。类似于MEMORY_ONLY_SER_2.
- 与RDD不同的是，DStream中默认的持久化级别，统一都是要序列化的。

### CheckPoint机制概述1

 - 实时计算程序的特点就是持续不断的对数据进行计算

 - 对实时计算应用的要求是必须要能够对与应用程序逻辑无关的失败，进行容错。

 - 要实现要求，Spark Streaming程序必须将足够的信息checkpoint到容错的存储系统上，从而让它从失败中进行回复。有两种数据需要被进行checkpoint：

    - 1 元信息checkpoint——定义了流式计算逻辑的信息，保存到容错的存储系统上，比如HDFS。当运行Spark Streaming应用程序的Driver进程所在节点失败时，该信息可用于进行恢复。元信息包括了：
       - 1配置信息——创建Spark Streaming应用程序的配置信息，比如SparkConf中的信息。
       - 2DStream的操作信息——定义了Spark Streaming应用程序的计算逻辑的Dstream操作信息。
       - 3未处理的batch信息——那些job正在排队，还没处理的batch信息。
   - 2 数据checkpoint——将实时计算过程中产生的RDD的数据保存到可靠的存储系统中。
     - 对于一些将多个batch数据进行聚合的，有状态的transformation操作，这是非常有用的。在这种transformation操作中，生成的RDD是依赖于之前的的RDD的，这会导致随着时间的推移，RDD的依赖链条变得越来越长。
     - 依赖链条越来越长，导致失败恢复时间越来越长。可以将有状态的transformation操作执行过程中产生的RDD，定期地被checkpoint到可靠的存储系统上，比如HDFS，从而削减RDD的依赖链条，进而缩短失败恢复时，RDD的恢复时间。
     - 元数据checkpoint主要是为了从driver失败中进行恢复；而RDD checkpoint主要是为了使用到有状态的transformation操作时，能够在其生产出的数据丢失时，进行快速的失败恢复。

- 何时启用CheckPoint机制

  - 1使用了有状态的transformation操作——比如updateStateByKey，或者reduceByKeyAndWindow操作被使用了，那么checkpoint目录要求是必须提供的，也就是说必须开启checkpoint机制，从而进行周期性的RDD checkpoint。
  - 2 要保证可以从Driver失败中进行恢复——元数据checkpoint需要启用来进行这种情况的恢复。
  - 3并不是所有的Spark Streaming应用程序都需要启用checkpoint机制，如果即不强制要求从Driver失败中自动进行恢复，又没使用有状态的transformation操作，那么就不需要启用checkpoint。事实上，这么做反而是有助于提升性能的。

- 如何启用CheckPoint机制

  - 1对于有状态的transformation操作，启用checkpoint机制，定期将其生产的RDD数据checkpoint，是比较简单的。

    - 可以通过配置一个容错的、可靠的文件系统(比如HDFS)的目录,来进行checkpoint机制，checkpoint数据就会写入该目录。使用StreamingContext的checkpoint()方法即可。

  - 如果为了要从Driver失败中进行恢复，那么启用checkpoint机制是比较复杂的。需要改写Spark Streaming应用程序。

    - 当应用程序第一次启动的时候，需要创建一个新的StreamingContext，并且调用其start()方法，进行启动。当Driver从失败中恢复过来时，需要从checkpoint目录中记录的元数据中恢复出来一个StreamingContext。

    - ```scala
      def functionToCreateContext():StreamingContext={
          val ssc= new StreamingContext(...)
          ssc.checkpoint(checkpointDirectory)
          ...
          ssc
      }
      val ssc= StreamingContext.getOrCreate(checkpointDirectory,functionToCreateContext _)
      ssc.start()
      ssc.awaitTermiation()
      ```

  - 配置spark-submit提交参数

    - 进行Spark Streaming应用程序的重写后，当第一次运行程序时，如果发现checkpoint目录不存在，那么就使用定义的函数来第一次创建一个StreamingContext，并将其元数据写入checkpoint目录；当从Driver失败中恢复过来时，发现checkpoint目录以及存在了，那么会使用该目录中的元数据创建一个StreamingContext。
    - 上面的重写时实现Driver失败自动恢复的第一步。第二步是必须确保Driver在失败时，自动被重启。
    - 要能够自动从Driver失败中恢复过来，运行Spark Streaming应用程序的集群，就必须及那块Driver运行的过程，并且在它失败时将它重启。对于Spark自身的standalone模式，就需要进行一些配置去supervise driver，在它失败时将其重启。
    - 首先，要在spark-submit中，添加--deploy-mode参数，默认其值为client，即在提交应用的机器上启动Driver；但是，要能够自动重启Driver，就必须将其值设置为cluster；此外需要添加supervise。
    - 使用上述步骤提交应用之后，就可以让driver在失败时自动被重启，并且通过checkpoint目录的元数据恢复StreamingContext。

- CheckPoint说明

  - 将RDD checkpoint到可靠的存储系统上，会耗费很多性能。当RDD被checkpoint时，会导致这些batch的处理时间增加。因此，checkpoint的间隔需要谨慎的设置。对于那些间隔很多的batch，比如1秒，如果还有执行checkpoint操作，则会大幅度削减吞吐量。而另外一方面如果checkpoint操作执行的太不频繁，就会当值RDD的lineage变长，又会有失败恢复时y间过长的风险。
  - 对于那些要求checkpoint的有状态的transformation操作，默认的checkpoint间隔通常是batch间隔的数倍，至少是10秒。使用DStream的checkpoint()方法，可以设置这个DStream的checkpoint的间隔时长。通常来说，将checkpoint间隔设置为窗口操作的滑动间隔的5~10倍，是个不错的选择。



## 程序部署、升级和监控

 - 部署应用程序
    - 1 有一个集群资源管理器，比如standalone模式下的Spark集群，Yarn模式下的Yarn集群等。
    - 2 打包应用程序为一个jar包。
    - 3为executor配置充足的内存，因为Receiver接收到的数据，是要存储在Executor的内存中的，所以Executor必须配置足够的内存来保存接受到的数据。要注意的是，如果你执行的窗口长度为10分钟的窗口操作，那么Executor的内存资源就必须足够保存10分钟内的数据，因此内存的资源要求是取决你执行的操作的
    - 4 配置checkpoint，如果你的应用程序要求checkpoint操作那么就必须配置一个Hadoop兼容的文件系统比如（HDFS）的目录作为checkpoint目录。
    - 5 配置dirver的自动恢复，如果要让driver能够在失败时自动恢复，一方面要重写driver程序，另一方面要在spark-submit中添加参数。
- 部署应用程序：启用日志预写系统
  - 预写日志机制，简写为WAL，全称为Write Ahead Log。从Spark 1.2版本开始，就引入了基于容错的文件系统的WAL机制。如果启用该机制，Receiver接受到的所有数据都会被写入配置的checkpoint目录中的预写日志。这种机制可以让driver在恢复的时候，避免数据丢失，并且可以确保整个实时计算过程中，零数据丢失。
  - 要配置该机制，首先调用SteamingContext的checkpoint()方法设置一个checkpoint目录。然后需要将spark.streaming.receiver.writeAheadLog.enable参数设置为true。
  - 然而，这种极强的可靠性机制会导致Receiver的吞吐量大幅度下降，因为单位时间内，有相当一部分时间需要将数据写入预写日志。如果又希望开启预写日志机制，确保数据零丢失，又不影响系统的吞吐量，那么可以创建多个输入DStream，启动多个Receiver。
  - 此外，在启用了预写日志机制之后，推荐将复制持久化机制禁用掉，因为所有数据以及保存在容错的文件系统中了，不需要再用复制机制进行持久化保存一份副本了。只要将输入DStream的持久化机制设置一下即可，persist(StorageLevel.MEMORY_AND_DISK_SER).

### 升级应用程序

 - 两种方式
    - 1 升级后的Spark应用程序直接启动，与旧的Spark应用程序并行执行，当确保新的应用程序启动没问题之后，就可以将旧的应用程序给停掉。但是要注意的是，这种方式只适用于，能够允许多个客户端读取各自独立的数据，也就是读取相同的数据。
    - 2小心地关闭已经在运行的应用程序，使用StreamingContext的stop()方法，可以确保接收到的数据都处理完之后才停止。然后将升级后的程序部署上去，启动。这样，就可以确保中间没有数据丢失和未处理。因为新的应用程序会从老的应用程序未消费到的地方，继续消费。但是注意，这种方式必须是支持数据缓存的数据源才可以，比如Kafka等。如果数据源不支持数据缓存，那么会导致数据丢失。
- 注意：配置了driver自动恢复机制时，如果想要根据旧的应用程序的checkpoint信息，启动新的应用程序，是不可行的。需要让新的应用程序针对新的checkpoint目录启动，或者删除之前的checkpoint目录。



### 监控应用程序

 - 当Spark Streaming应用启动时，Spark Web UI会显示一个独立的streaming tab，会显示Receiver的信息，比如是否活跃，接收到了多少数据，是否有异常等；还会显示完成batch的信息，batch的处理时间、队列延迟等。这些信息可以用于监控spark streaming应用的进度。

 - Spark UI中一下两个统计指标格外重要

    - 1、处理时间——每个batch的数据的处理的耗时
    - 2、调度延时——一个batch在独队列中阻塞住，等待上一个batch完成物理的时间。

- 如果batch的处理时间，比batch的间隔时间要长，而且调度延迟时间持续增长，应用程序不足以使用当前设定的速率来处理接收到的数据，此时可以考虑增加batch的间隔时间。

  

## 容错机制和事务语义详解

### 容错机制的背景

 - 要理解Spark Streaming提供的容错机制，先回忆一下Spark RDD的基础容错语义：
    - 1、RDD，Ressilient Distributed Dataset，是不可变的、确定的、可重新计算的、分布式的数据集。每个RDD都会记住确定好的计算操作的血缘关系，（val lines = sc.textFile(hdfs file); val words = lines.flatMap(); val pairs = words.map(); val wordCounts = pairs.reduceByKey()）这些操作应用在一个容错的数据集上来创建RDD。
    - 2、如果因为某个Worker节点的失败（挂掉、进程终止、进程内部报错），导致RDD的某个partition数据丢失了，那么那个partition可以通过对原始的容错数据集应用操作血缘，来重新计算出来。
    - 3、所有的RDD transformation操作都是确定的，最后一个被转换出来的RDD的数据，一定是不会因为Spark集群的失败而丢失的。
 - Spark操作的通常是容错文件系统中的数据，比如HDFS。因此，所有通过容错数据生成的RDD也是容错的。然而，对于Spark Streaming来说，这却行不通，因为在大多数情况下，数据都是通过网络接收的。要让Spark Streaming程序中，所有生成的RDD，都达到与普通Spark程序的RDD相同的容错性，接收到的数据必须被复制到多个Worker节点上的Executor内存中，默认的复制因子是2。
 - 基于上述理论，在出现失败的事件时，有两种数据需要被恢复：
    - 1、数据接收到了，并且已经复制过——这种数据在一个Worker节点挂掉时，是可以继续存活的，因为在其他Worker节点上，还有它的一份副本。
    - 2、数据接收到了，但是正在缓存中，等待复制的——因为还没有复制该数据，因此恢复它的唯一办法就是重新从数据源获取一份。
 - 此外，还有两种失败是我们需要考虑的：
    - 1、Worker节点的失败——任何一个运行了Executor的Worker节点的挂掉，都会导致该节点上所有在内存中的数据都丢失。如果有Receiver运行在该Worker节点上的Executor中，那么缓存的，待复制的数据，都会丢失。
    - 2、Driver节点的失败——如果运行Spark Streaming应用程序的Driver节点失败了，那么显然SparkContext会丢失，那么该Application的所有Executor的数据都会丢失。

### Spark Streaming容错语义的定义

 - 流式计算系统的容错语义，通常是以一条记录能够被处理多少次来衡量的。有三种类型的语义可以提供：
    - 1、最多一次：每条记录可能会被处理一次，或者根本就不会被处理。可能有数据丢失。
    - 2、至少一次：每条记录会被处理一次或多次，这种语义比最多一次要更强，因为它确保零数据丢失。但是可能会导致记录被重复处理几次。
    - 3、一次且仅一次：每条记录只会被处理一次——没有数据会丢失，并且没有数据会处理多次。这是最强的一种容错语义。

### Spark Streaming的基础容错语义

 - 在Spark Streaming中，处理数据都有三个步骤：
    - 1、接收数据：使用Direct方式接收数据。
    - 2、计算数据：使用DStream的transformation操作对数据进行计算和处理。
    - 3、推送数据：最后计算出来的数据会被推送到外部系统，比如文件系统、数据库等。
 - 如果应用程序要求必须有一次且仅一次的语义，那么上述三个步骤都必须提供一次且仅一次的语义。每条数据都得保证，只能接收一次、只能计算一次、只能推送一次。Spark Streaming中实现这些语义的步骤如下：
    - 1、接收数据：不同的数据源提供不同的语义保障。
    - 2、计算数据：所有接收到的数据一定只会被计算一次，这是基于RDD的基础语义所保障的。即使有失败，只要接收到的数据还是可访问的，最后计算出来的数据一定是相同的。
    - 3、推送数据：output操作默认能确保至少一次的语义，因为它依赖于output操作的类型，以及底层系统的语义支持（比如是否有事务支持等），用户可以实现它们自己的事务机制来确保一次且仅一次的语义。

### 接收数据的容错语义

 - 1、基于文件的数据源

    - 如果所有的输入数据都在一个容错的文件系统中，比如HDFS，Spark Streaming一定可以从失败进行恢复，并且处理所有数据。这就提供了一次且仅一次的语义，意味着所有的数据只会处理一次。
- 2、基于Direct的数据源
    - 	Kafka Direct API，可以保证，所有从Kafka接收到的数据，都是一次且仅一次。基于该语义保障，如果自己再实现一次且仅一次语义的output操作，那么就可以获得整个Spark Streaming应用程序的一次且仅一次的语义

### 输出数据的容错语义

 - output操作，比如foreachRDD，可以提供至少一次的语义。那意味着，当Worker节点失败时，转换后的数据可能会被写入外部系统一次或多次。对于写入文件系统来说，这还是可以接受的，因为会覆盖数据。但是要真正获得一次且仅一次的语义，有两个方法：
    - 1、幂等更新：多次写操作，都是写相同的数据，例如saveAs系列方法，总是写入相同的数据。
    - 2、事务更新：所有的操作都应该做成事务的，从而让写入操作执行一次且仅一次。给每个batch的数据都赋予一个唯一的标识，然后更新的时候判定，如果数据库中还没有该唯一标识，那么就更新，如果有唯一标识，那么就不更新。
      dstream.foreachRDD { (rdd, time) =>
        rdd.foreachPartition { partitionIterator =>
          val partitionId = TaskContext.get.partitionId()
          val uniqueId = generateUniqueId(time.milliseconds, partitionId)
          // partitionId和foreachRDD传入的时间，可以构成一个唯一的标识
        }
      }



## 性能调优

### 数据接收并行调优

 - 显式地对输入数据流进行重分区。使用inputStream.repartition(<number of partitions>)即可。这样就可以将接收到的batch，分布到指定数量的机器上，然后再进行进一步的操作。
 - 什么时候会用
    - 针对分区过少的时候，可以提高分区数据，例如 distinct之后，可以通过repartition增加分区数量
    - 针对分区过多的时候，可以减少分区数据量，例如fliter之后，每个分区的数据量都变少了，所以可以合并分区，也是通过reparation重新指定分区数量即可

### 数据处理并行度调优

- 如果在计算的任何stage中使用的并行task的数量没有足够多，那么集群资源是无法被充分利用的。举例来说，对于分布式的reduce操作，比如reduceByKey和reduceByKeyAndWindow，默认的并行task的数量是由spark.default.parallelism参数决定的。你可以在reduceByKey等操作中，传入第二个参数，手动指定该操作的并行度，也可以调节全局的spark.default.parallelism参数。

### 数据序列化调优

  - 数据序列化造成的系统开销可以由序列化格式的优化来减小。在流式计算的场景下，有两种类型的数据需要序列化。
      - 1、输入数据：默认情况下，接收到的输入数据，是存储在Executor的内存中的，使用的持久化级别是StorageLevel.MEMORY_AND_DISK_SER_2。这意味着，数据被序列化为字节从而减小GC开销，并且会复制以进行executor失败的容错。因此，数据首先会存储在内存中，然后在内存不足时会溢写到磁盘上，从而为流式计算来保存所有需要的数据。这里的序列化有明显的性能开销——Receiver必须反序列化从网络接收到的数据，然后再使用Spark的序列化格式序列化数据。
      - 2、流式计算操作生成的持久化RDD：流式计算操作生成的持久化RDD，可能会持久化到内存中。例如，窗口操作默认就会将数据持久化在内存中，因为这些数据后面可能会在多个窗口中被使用，并被处理多次。然而，不像Spark Core的默认持久化级别，StorageLevel.MEMORY_ONLY，流式计算操作生成的RDD的默认持久化级别是StorageLevel.MEMORY_ONLY_SER ，默认就会减小GC开销。
- 在上述的场景中，使用Kryo序列化类库可以减小CPU和内存的性能开销。使用Kryo时，一定要考虑注册自定义的类，并且禁用对应引用的tracking（spark.kryo.referenceTracking）。
  - 跟踪对同一个对象的引用情况，这对发现有循环引用或同一对象有多个副本的情况是很有用的。设置为false可以提高性能
  - 在一些特殊的场景中，比如需要为流式应用保持的数据总量并不是很多，也许可以将数据以非序列化的方式进行持久化，从而减少序列化和反序列化的CPU开销，而且又不会有太昂贵的GC开销。举例来说，如果你数秒的batch interval，并且没有使用window操作，那么你可以考虑通过显式地设置持久化级别，来禁止持久化时对数据进行序列化。这样就可以减少用于序列化和反序列化的CPU性能开销，并且不用承担太多的GC开销。

### batch interval调优

- 如果想让一个运行在集群上的Spark Streaming应用程序可以稳定，它就必须尽可能快地处理接收到的数据。换句话说，batch应该在生成之后，就尽可能快地处理掉。对于一个应用来说，这个是不是一个问题，可以通过观察Spark UI上的batch处理时间来定。batch处理时间必须小于batch interval时间。
- 基于流式计算的本质，batch interval对于，在固定集群资源条件下，应用能保持的数据接收速率，会有巨大的影响。例如，在WordCount例子中，对于一个特定的数据接收速率，应用业务可以保证每2秒打印一次单词计数，而不是每500ms。因此batch interval需要被设置得，让预期的数据接收速率可以在生产环境中保持住。
- 为你的应用计算正确的batch大小的比较好的方法，是在一个很保守的batch interval，比如5~10s，以很慢的数据接收速率进行测试。要检查应用是否跟得上这个数据速率，可以检查每个batch的处理时间的延迟，如果处理时间与batch interval基本吻合，那么应用就是稳定的。否则，如果batch调度的延迟持续增长，那么就意味应用无法跟得上这个速率，也就是不稳定的。因此你要想有一个稳定的配置，可以尝试提升数据处理的速度，或者增加batch interval。记住，由于临时性的数据增长导致的暂时的延迟增长，是合理的，只要延迟情况可以在短时间内恢复即可。

