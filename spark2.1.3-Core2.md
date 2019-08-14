# spark2.1.3-Core2

## RDD的基本特性

 - 1.RDD是Resilient Distributed DataSet.即弹性分布式数据集。
 - 2.RDD的数据是有多个分区的。
 - 3.每一个分区的数据存在于集群的某一个节点上，后期会对于一个Task任务来处理，多个分区的数据可以并行处理。
 - 4.RDD的容错性，当某个RDD的某一个partition对应的数据丢失了，那么Spark可以根据RDD的来源重新计算该patition的数据，Spark任务不需要重新执行。
 - 5.弹性，表示RDD的数据默认是存放在内存中，如果内存中存不下，也是可以存储到磁盘中的。

## Spark 任务的执行方式

 - 1.可以在本地执行setMaster("local")（直接在idea中执行-（不推荐））
 - 2.使用 spark-submit 提交jar包到yarn集群执行
 - 3.使用spark0shell 交互式命令行中执行

## Spark 架构中的概念

 - Driver

    - 我们编写的Spark 程序就在Driver（进程）上，由Driver进程负责执行
    - Driver进程所在的节点可以是Spark集群的某一个节点或者就是我们提交Spark程序的机器。

- Master

  - 集群的主节点中启动的进程。
  - 主要负责资源的管理与分配，还有集群的监控等。

- Worker

  - 集群的从节点中启动的进程
  - 主要负责启动其他进程来执行具体数据的处理和计算等。

- Executor

  - 是一个独立的进程
  - 此进程由Worker负责启动，主要为了执行数据处理和计算。

- Task

  - 是一个线程
  - 由Executor负责启动，真正干活的模块。

- Spark 程序执行流程

  ![Spark执行流程](https://raw.githubusercontent.com/wangxiaolin123/bigData/master/img/Spark执行流程.png)


# 创建RDD的三种方式

 - 1.使用集合创建【测试和开发代码的时候使用】
 - 2.使用本地文件创建【临时用】
 - 3.使用hdfs文件创建【生产环境使用】
    - 默认根据hdfs文件获取的RDD分区数量和hdfs中文件的block数量有关系，默认是一 一对应的。

# transformation 和 action 算子介绍

 -	transformation算子一般基于RDD做一些处理，处理之后返回新的RDD。
 -	action 算子是产生最终结果的算子，返回的数据不是RDD，如果有返回数据，那么数据返回到Driver端。
 -	transformation 算子有lazy这个特性（延迟加载），如果一个sparkjob中只有transformation算子，没有action算子，那么这个任务是不会执行的。
 -	action 算子才会触发sparkjob的执行，从而触发这个action之前所有的transformation的执行。
 -	由于transformation算子的lazy特性，Spark通过这种lazy特性，来进行底层Spark应用执行的优化，避免产生过多中间结果。
 -	一些特殊的transformation算子：groupByKey,sortByKey,reduceByKey,这些算子只能在Tuple2（）类型的RDD后面使用。

# 常见的transformation算子

- map：将RDD中中每个元素传入自定义函数，获取一个新函数组成新的RDD
- filter：对RDD中每个元素进行判断，如果返回true则保留，返回false则剔除。
- flatMap:与map类似，但是对每个元素都可以返回一个或多个新元素。
- groupByKey：根据key进行分组，每个key对应一个Iterable<value>
- reduceByKey:对每个key对应的value进行reduce操作。
- sortByKey：对每个key对应的value进行排序操作。
- join:对两个包含<key,value>对的RDD进行join操作，每个key join 上的pair，都会传入自定义函数进行处理。
- cogroup:同join，但是每个key对应的Iterable<value>都会传入自定义函数进行处理。

# 常见的action 算子



 - reduce：将RDD中的所有元素进行聚合操作。第一个和第二个元素聚合，值与第三个元素聚合，值与第四个元素聚合，依次类推。

 - collect:将RDD中所有元素获取到本地客户端。

 - count：获取RDD元素总数。

 - 获取RDD中前n个元素。

 - saveAsTextFile：将RDD元素保存到文件中，对每个元素调用toString方法。

 - countByKey：对每个key对应的值进行count计数。

 - foreach：遍历RDD中的每个元素。

   

# RDD持久化详解

## RDD持久化原理

 - Spark 非常重要的一个特性是可以将RDD持久化在内存中。当对RDD执行持久化操作时，每个节点都会将自己操作的RDD持久化到内存中，并且在之后对RDD的反复使用中，直接使用内存缓存的partition。这样的话，对于针对一个RDD反复执行多个操作的场景，就只要对RDD计算一次即可，后面直接使用该RDD，而不需要反复计算多次该RDD。

 - 巧妙使用RDD持久化，甚至在某些场景下，可以将Spark应用程序的性能提升10倍。对于迭代式算法和快速交互式应用来说，RDD持久化，是非常重要的。

 - 要持久化一个·RDD，只要调用其cache()或者persist()方法即可。该RDD第一次被计算出来时，就会直接缓存在每个节点中。而且Spark 的持久化机制还是自动容错的，如果持久化的RDD的任何partition丢失了，那么Spark会自动通过其源RDD，使用transformation操作重新计算该partition。

 - cache()和persisit()的区别在于chache（）是persistent()的一种简化方式，cache()的底层就是调用persisit()的无参版本，同时就是调用persisit(MEMORY_ONLY)，将数据持久化到内存中。如果需要从内存中清除缓存，那么可以使用unpersisit()方法。

 - Spark自己也会在shuffle操作时，进行数据的持久化，比如写入磁盘，主要是为了在节点失败时，避免要重新计算整个过程。

   

## RDD持久化策略

| 持久化级别  | 含义 |
| :--- | -------- |
| MEMORY_INLY | 以非序列化的java对象方式持久化在JVM内存中。如果内存无法完全存储RDD所有的partition，那么那些没有持久化的partition就会在下一次需要使用它的时候，重新被计算。 |
| MEMORY_AND_DISK | 同上，但是当某些partition无法存储在内存中时，会持久化到磁盘中。下次需要使用这些partition时，需要从磁盘上读取。 |
| MEMORY_ONLY_SER | 同MEMORY_ONLY，但是会使用java序列化方式，将java对象序列化之后进行持久化。可以减少内存开销，但是需要进行反序列化，因此会加大cpu开销。 |
| MEMORY_AND_DSIK_SER | 同MEMOEY_AND_DISK。但是使用序列化方式持久化java对象。 |
| DISK_ONLY | 使用非序列化方式持久化，完全存储到磁盘上。 |
| MEMORY_ONLY_2 ,MEMORY_AND_DISK_2等 | 如果是尾部加2的持久化级别，表示会将持久化数据复用一份，保存到其他节点，从而在数据丢失时，不需要再次进行计算，只需要使用备份数据即可。 |



## 如何选择RDD持久化策略

 - Spark 提供的多种持久化级别，主要时为了在CPU和内存消耗之间进行取舍。
 - 下面是一些通用的持久化级别的选择建议。
    - 1 优先使用MEMORY_ONLY,如果可以缓存所有数据的话，那么就是使用这种策略。因为纯内存速度最快，而且没有序列化，不需要消耗cpu进行反序列化操作。
    - 2 如果MEMORY_ONLY策略无法存储下所有数据的话，那么使用MEMORY_ONLY_SER，将数据进行序列化存储，纯内存操作还是非常快，只是要消耗cpu进行反序列化。
    - 3 如果需要进行快速的失败恢复，那么就选择带后缀为_2的策略，进行数据的备份，这样在失败时，就不需要重新计算了。
    - 4 能不使用DISK相关的策略，就不使用，有的时候，从磁盘读取数据还不如重新计算一次。



# 共享变量

## 共享变量的工作原理

 - Spark一个非常重要的特性就是共享变量
    - 

## Broadcast Variable 和Accumulator

