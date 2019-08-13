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
 -	action 算子才会触发sparkjob的执行。
 -	由于transformation算子的lazy特性，Spark通过这种lazy特性，来进行底层Spark应用执行的优化，避免产生过多中间结果。
 -	一些特殊的transformation算子：groupByKey,sortByKey,reduceByKey,这些算子只能在Tuple2（）类型的RDD后面使用。

# 常见的transformation算子

- 

# 常见的action 算子

 - 
 - 

