# Spark 高级功能


## 宽依赖和窄依赖

- 窄依赖（Narrow Dependency）:  **指父RDD的每个分区只被子RDD的一个分区所使用** 例如map、filter、union等操作会产生窄依赖
  - 一个RDD对它的父RDD只有简单的一对一的关系，也就是说，RDD的每个partition仅仅依赖于父RDD中的一个partition，父RDD和子RDD的partition之间的对应关系，是一对一的。
- 宽依赖（Shuffle Dependency）：**父RDD的每个分区都可能被子RDD的多个分区所使用。** 例如groupByKey、reduceByKey、sortByKey等操作会产生宽依赖，会产生shuffle。
  - 也就是说，每一个父RDD的partition中的数据都可能会传输一部分到下一个RDD的每个partition中。此时会出现父RDD和子RDD的partition之间，具有错综复杂的关系，那么这种情况就叫做两个RDD之间是宽依赖，同时他们之间会发生shuffle操作。

## Stage 的划分

- Spark 的job是根据action算子触发的，遇到action算子就会起一个job，job里面又划分了很多stage，然后每一个stage里面运行了很多个task，**satge的划分依据就是看是否产生了shuffle（即宽依赖）** 遇到一个shuffle操作就划分为前后两个stage，stage是由一组并行的task组成，stage会将一批task用TaskSet封装，提交给TaskScheduler进行分配，最后发送到Executor执行。 
- 切割规则：从后往前，遇到宽依赖就切割stage
  - ![stage的划分](https://raw.githubusercontent.com/wangxiaolin123/bigData/master/img/SparkStage.png)
  - 从后往前看，RDD之间是有血缘关系的，后面的RDD依赖前面的RDD，也就是说，后面的RDD要等前面的RDD执行完，才会执行，所有从后往前遇到宽依赖就分为两个stage，shuffle前一个，shuffle后一个，如果整个过程没有产生shufle那就指挥有一个stage，所以上图中 RDD a就单独一个stage1，RDD c,d,e,f被划分在stage2中，最后RDD b和RDD g划分在了stage3里面。

## Spark 的三种提交模式

 - 1 第一种模式，standalone模式，基于Spark自己的Master-Worker集群，
    - 指定	--master spark://hadoop128:7077
- 2 第二种，基于Yarn的client模式
  - 指定	--master yarn --deploy-mode client
  - 这种方法主要用于测试，因为driver程序运行在本地客户端，dirver负责调度job，会与yarn集群产生大量的通信，从而导致网卡流量激增，并且当我们执行一些action操作的时候数据也会返回driver端，driver端的机器配置一般都不高，可能会导致内存溢出等问题。
- 3 第三种，基于Yarn的cluster模式。 【推荐】
  - 指定	--master yarn --deploy-mode cluster
  - 这种方式driver程序运行在集群中的某一台机器上，不会村咋网卡流量激增的问题，并且机器的配置回避普通客户端的机器高。

## Shuffle原理剖析

 - 在Spark中，什么情况下会发送shuffle？

    - reduceByKey、groupByKey、sortByKey、countByKey、join、cogroup等操作。

- 默认的shuffle操作原理剖析

  - ![SparkShuffle-1](https://raw.githubusercontent.com/wangxiaolin123/bigData/master/img/SparkShuffle-1.png)

- 优化后的shuffle操作原理剖析

  - ![SparkShuffle-2](https://raw.githubusercontent.com/wangxiaolin123/bigData/master/img/SparkShuffle-2.png)

- **Spark shuffle操作的两个特点**

  - 第一个特点：

    - 在Spark早期版本中，bucket缓存时非常重要的，因为需要将一个shuffleMapTask所有数据都写入内存缓存之后，才会刷新到磁盘。但是这就有一个问题，如果map side数据过多，那么很容易造成内存移除。所以Spark 在新版本中优化了，默认内存缓存是100kb，写入一点数据达到了刷新到磁盘的阈值之后，就会将数据一点一点地刷新到磁盘。
    - 这种操作的优点是不容易发生内存溢出。缺点在于，如果内存缓存过小，那么可能发生过多的磁盘写IO操作。所以这里的内存缓存太小，是可以根据实际的业务情况进行优化的。

  - 第二个特点：

    - 与MapReduce完全不一样的是，MapReduce它必须将所有数据写入本地磁盘文件以后，才能启动reduce操作，来拉取数据。为什么？因为mapreduce要实现默认的根据key的排序，所有要排序，肯定得写完所有数据，才能排序，然后根据educe来拉取。
    - 但是Spark不需要，Spark默认情况下是不会对数据进行排序的。因此shufflemapTask没写入一点数据，ResultTask就可以拉取一点数据，然后在本地执行我们定义的聚合函数和算子，进行计算。
    - Spark 这种机制的好处在于速度比MapReduce快多了。但是也有一个问题，MapReduce提供的reduce，是可以处理每个key对应的value上的数据，很方便。但是Spark这种实时拉取机制，提供不了直接处理key对应的values的算子，只能通过groupByKey，先shuffle，然后用map算子来处理每个key对应的values。就没有MapReduce的计算模型这么方便。

    

## CheckPoint原理剖析

 - CheckPoint是什么

    - checkpoint 是Spark提供的一个比较高级的功能。有的时候，我们的Spark应用程序，特别复杂，从初始的RDD开始，到最后完成可能需要很长时间，比如超过20个transformation操作，整个应用运行时间也特别长。
    - 在上述情况下，就比较适合使用checkpoint功能。因为对于特别复杂的Spark应用，有很高的风险，会出现某个要反复使用的RDD因为节点的故障导致丢失，虽然之前持久化过，但还是导致数据丢失了。换一种说法；出现失败时没有容错机制，当后面的transformation操作又要使用到该RDD时，就会发现数据丢失了，此时如果没有进行容错处理的话，那么可能就又要重新计算一次数据。
    - 针对上述情况，整个Spark应用程序的容错性很差。

 - CheckPoint的功能

    - 对于一个复杂的RDD chain，如果我们担心中间某些关键的，在后面会反复几次使用的RDD，可能会因为节点的故障，导致持久化数据的丢失，那么就可以针对该RDD格外启动checkpoint机制，实现容错和高可用。
    - checkpoint首先调用SparkContext的setCheckPintDir（）方法，设置一个容错的文件系统的目录，比如说HDFS；然后对RDD调用checkpoint()方法。之后在RDD所处的job运行结束之后，会启动一个单独的job，来将checkpoint过来的RDD的数据写入之前设置的文件系统，进行高可用、容错的类持久化操作。
    - 那么此时，即使在后面使用RDD时，它的持久化的数据，不小心丢失了，但是还可以从它的checkpoint文件直接读取其数据，而不需要重新计算。（CacheManager）

 - CheckPoint原理

    - 如何进行checkpoint？

       - SparkContext.setCheckpointDir()
       - RDD.checkpoint()

   - Checkpoint与持久化的不同

     - 持久化，只是将数据保存在内存里，RDD的lineage(依赖关系）是不变的
     - 但是checkpoint执行后，RDD就没有之前所谓lineage(依赖关系)RDD了，也就是说它的lineage改变了。
     - 持久化的数据丢失的可能性较大
     - checkpoint的数据通常保存在高可用的文件系统中（HDFS）,所以丢失的可能性很低

     要checkpoint的RDD，限制性persist（Storage Level.DISK_ONLY)操作

    - 

   