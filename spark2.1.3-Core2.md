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