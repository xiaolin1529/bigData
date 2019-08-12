# spark

## spark简介

​	spark是一种通用的大数据计算框架，和传统大数据技术Hadoop的MapReduce引擎，以及Storm流式实时计算引擎等。

​	Spark包含了大数据领域常见的各种计算框架

   - Spark Core用于流式计算
   - spark SQL 用于交互式查询
   - Spark Streaming 用于实时流式计算
   - Spark MLlib 用于机器学习
   - Spark GraphX 用于图计算

Spark主要用于大数据计算，Hadoop以后只要用于大数据的存储（HDFS，HBase）等，以及资源管理（Yarn）

在实际工作中会把Spark+Hadoop结合起来使用。

Spark 是一种“One Stack to rule them all” 的大数据计算框架，期望使用一个技术栈就完美解决大数据领域的各种计算任务。Apache 官方定义Spark：通用的大数据快速处理引擎。

Spark 使用 Spark RDD、Spark SQL、Spark Streaming、MLlib、GraphX 成功解决大数据领域中 离线批处理、交互式查询、实时流计算、机器学习、图计算等最重要的任务与问题。

Spark 除了一站式的特点外，另一个最重要的特点，就是基于内存进行计算，从而让它的速度可以达到MapReduce的数倍甚至数十倍。



# Spark 整体架构

![Spark 整体架构](https://raw.githubusercontent.com/wangxiaolin123/bigData/master/img/Spark整体架构.png)

# Spark 特点

 + 速度快
    + Spark是基于内存进行计算（有部分内容基于磁盘，比如shuffle）
+ 容易开发上手
  + Spark 基于RDD的计算模型，比Hadoop基于MapReduce的计算模型更加易于理解，更加利于上手开发，实现各种复杂功能，如二次排序、topN等复杂操作时，更加简便。
+ 超强的通用性
  + Spark 提供Spark RDD、Spark SQL、Spark Streaming、Spark MLlib、Spark GraphX等技术组件，可以一站式地完成大数据领域的离线批处理、交互式查询、实时流式计算、机器学习、图计算等常见任务。
+ 集成Hadoop
  + Spark与Hadoop进行了高度的集成，两者可以完美配合使用。hadoop的HDFS、Hbase负责存储，Yarn负责资源管理；Spark 负责大数据计算，实际上Spark+Hadoop是一个非常完美的组合。



# Spark vs 其他框架

## Spark  vs MapReduce

 - MapReduce能够完成的各种离线批处理功能、以及常见算法（二次排序、topN）等，基于Spark RDD的核心编程，都可以实现，并且可以 更好地、更容易地实现 。而且基于Spark RDD编写的离线批处理程序，运行速度都是MapReduce的数倍，速度上有非常明显的优势。
 - Spark相较于MapReduce速度快的最主要原因是 MapReduce的计算模型太死板，必须是Map-Reduce模式，有时候完成一些过滤之类的操作，也要经历Map-Reduce过程，这样就必须经过shuffle过程。而MapReduce的shuffle过程是最消耗性能的，因为shuffle中间的操作必须基于磁盘来读写。虽然Spark的shuffle虽然也要基于磁盘，但是大量的transformation操作，比如单纯的map或filter操作，可以直接基于内存进行pipeline操作，速度性能自然大大提升。
 - Spark也要劣势。由于Spark基于内存进行计算，虽然开发容易，但是真正面对大数据的时候（比如一次操作针对10亿以上级别），在没有进行调优的情况下，可能会出现各种各样的问题，比如OOM内存溢出等等。导致Spark程序可能无法完全运行起来就保存挂掉了，而MapReduce即使运行缓慢，但是至少可以慢慢运行完。



## Spark SQL vs Hive

 - Spark SQL 实际不能完全替代Hive，因为Hive是基于HDFS的数据仓库，并且提供了基于SQL模型的分布式交互查询的查询引擎。
 - 严格来说 Spark SQL能够替代的是Hive的查询引擎，而不是Hive本身，实际上在生产环境下，SparkSQL，也是针对Hive数据仓库中的数据进行查询，Spark本身自己是不提供存储的，自然无法替代Hie作为数据仓库的这个功能
 - Spark SQL 相较于Hive查询引擎来说，就是速度快，同样的SQL语句，可能使用Hive的查询引擎，由于其底层基于MapReduce，必须经过shuffle过程走磁盘，因此速度是非常缓慢的。很多复杂的SQL语句，在hive中执行都需要一个小时以上的时间。而Spark SQL由于其底层基于Spark自身的基于内存的特点，因此速度达到了Hive查询引擎的数倍以上。
 - Spark SQL相较于Hive的另一个优点，就是支持大量不同的数据源，包括hive、json、parquet、jdbc等等。此外Spark SQL由于身处Spark技术堆栈内，也是基于RDD来工作，因此可以与Spark的其他组件无缝整合使用，配合起来实现许多复杂的功能。比如Spark SQL 支持可以直接针对hdfs文件执行SQL语句。

## Spark Streaming vs Storm

 - Spark Streaming 和Storm 都可以进行实时流计算。但是他们两者区别是非常大的。Spark Streaming 和Storm的计算模型完全不一样，Spark Streaming是基于RDD的，因此需要将一小段时间内的，比如1秒内的数据收集起来，作为一个RDD，然后针对这个batch的数据进行处理。而Storm却可以做到每来一条数据，都可以立即进行处理和计算。因此，Spark Streaming实际上严格来说，只能称作准实时流计算框架；而Storm是真正意义上的实时计算框架。
 - Storm 支持在分布式流式计算程序运行过程中动态调整并行度，从而动态提高并发处理能力。而Spark Streaming无法动态调整并行度。
 - Spark Streaming 基于batch进行处理，相较于Storm基于单挑数据进行处理，具有数倍甚至数十倍的吞吐量
 - Spark Streaming 由于身处Spark生态圈内，因此Spark Streaming可以与Spark Core、Spark SQL，甚至是Spark MLlib、Spark GraphX进行无缝整合。流式处理完的数据可以立即进行各种map、reduce转换操作，可以立即使用SQL进行查询，甚至可以立即使用机器学习或图计算算法进行处理。这种一站式的大数据处理功能是Storm无法匹敌的。
 - 综合来看，通常在对实时性要求比较高，而且实时数据量不稳定，比如在白天有高峰期的情况下，可以选择使用Storm。但是如果是对实时性要求一般，允许1秒的准实时处理，而且不要求动态调整并行度的话，选择Spark Streaming是最好的选择。

# Spark的应用场景

 - 首先 ，Spark相较于MapReduce来说，可以立即替代的，并且会产生非常理想的效果，就是要求低延时的复杂大数据交互式系统。可以用SPark Core 替代MapReduce
 - 其次，相对于Hive来水，对于某些需要根据用户选择的条件，动态拼接SQL语句，进行某类特定查询统计任务的系统，也要求低延时甚至希望达到几分钟之内。此时也可以使用Spark SQL 替代Hive查询引擎。此时使用Hive查询引擎执行一个复杂SQL可能需要几十分钟，而使用Spark SQL可能只需要使用几分钟就可以达到用户期望的效果。
 - 对于Storm来说，如果仅要求对数据进行简单的流式处理计算，那么选择torm或Spark都可以。但是如果需要对流式计算的中间结果（RDD），进行复杂的后续处理，则使用Spark更好，因为Spark本身提供了很多功能比如map、reduce、groupByKey、filter等等。