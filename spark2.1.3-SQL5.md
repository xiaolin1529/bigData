# spark2.1.3-SQL5

 - Spark SQL是Spark中的一个模块，主要用于进行结构化数据的处理。它提供的最核心的编程抽象，就是DataFrame.同时Spark SQL还可以作为分布式的SQL查询引擎。

## DataFrame的使用

 - DataFrame，可以理解为是，以列的形式组织的，分布式的数据集合，DataFrame=RDD+Schema。它其实和关系型数据库中的表非常类似，但是底层做了很多的优化。DataFrame可以通过很多源来进行构建。

    - 包括：结构化的数据文件，Hive中的表，外部的关系型数据库以及RDD。

- DataSet是分布式的数据集合。DataSet包含了DataFrame的功能，在Spark2。0中两者统一，DataFrame表示为DataSet[Row]

  - DataSet可以在编译时检查类型，DataFrame在运行时检查类型
  - DataSet是面向对象的编程接口。

- 创建SparkSession

  - 要使用Spark SQL首先创建SparkSession对象
  - Spark Session中包含了spark.sparkContext和spark.sqlContext
  - `val sparksql = SparkSession.builder().appName(appName).config(conf).getOrCreate()`
  - // 隐式转换，将RDD隐式转换为DataFrame
  - `import sparksql.implicits._`

- 创建DataFrame

  - 使用SparkSession，可以从RDD、Hive表或者其他数据源来创建一个DataFrame。使用json文件创建DataFrame
  - `val df =sparksql.read.json("hdfs://hadoop128:9000/students.json")`

- DataFrame的常见操作

  - Scala版本

  - ```scala
    val df = sparksql.read.json("hdfs://hadoop128:9000/students.json")
    df.show()
    df.printSchema()
    df.select("name").show()
    df.selec(df("name"),df("age")+1).show()
    df.filter(df("age")>21).show()
    df.groupBy("age").count().show()
    ```

- RDD 转换为DataFrame

  - RDD转换为DataFrame,可以直接针对Hdfs等任何可以构建RDD的数据，使用Spark SQL进行SQL查询。这个功能是无比强大的。

  - Spark SQL支持两种方式将RDD转换为DataFrame

    - 第一种，使用反射来推断包含了特定数据类型的RDD的元数据。这种基于反射的方式，代码比较简洁，当你已经指定你的RDD的元数据时，是一种非常不错的方式。
      - Scala版本：scala由于具有隐式转换的特性，所有Spark SQL的scala接口时支持自动将包含了case class的RDD转换为DataFrame的。case class就定义了元数据。Spark SQL会通过反射读取传递给case class的参数的名称，然后将其作为列名。
    - 第二种方式，通过编程接口创建DataFrame，你可以在程序运行时动态构建一份元数据，然后将其应用到已经存在的RDD上。这种方式的代码比较冗长，但是如果在编写程序时，还不知道RDD的元数据，只要在程序运行时才能动态得知其元数据，那么只能通过这种动态构建元数据的方式。
      - 当case clasee中的字段无法预先定义和知道的时候，比如要动态从一个文件中读取数据结构，那么就只能用编程方式动态指定元数据，首先从原始RDD创建一个元素为Row的RDD，其次创建一个StructType，来代表Row；最后将动态定义的元数据应用到RDD<Row>上。

    



## 数据源详解【parquet+json+jdbc】

 - 对于Spark SQL的DataFrame来说，无论时从什么数据源创造出来的DataFrame，都有一些共同的load和save操作。

    - load操作主要用于加载数据，创建出DataFrame
    - save操作主要用于将DataFrame中的数据保存到文件中。

- 如果不指定format，则默认读取的数据是parquet格式，也可以手动指定用来操作的数据源类型。数据源通常需要使用其全限定名来指定，比如parquet是org.apache.spark.sql.parquet。但是Spark SQL内置了一些数据源类型，比如json，parquet，jdbc，orc，csv，text。实际上通过这个功能，就可以在不同类型的数据源之间进行转换了。比如将json文件中的数据保存到parquet文件中。默认情况下，如果不指定数据源类型，那么就是parquet。

  - `val df = sparksql.read.format("json").load("students.json")`
  - `df.select("name","age").write.format("parquet").save("namesAndAges.parquet")`

- Spark SQL对于save操作，提供了不同的save mode。主要用来处理，当目标位置，已经有数据时，应该如何处理。而save操作并不会执行锁操作，并且不是原子的，因此有一定风险出现脏数据。

  | Save Mode                    | 意义                                                         |
  | ---------------------------- | ------------------------------------------------------------ |
  | SaveMode.ErrorIfExists(默认) | 如果目标位置已经存在数据，那么抛出异常                       |
  | SaveMode.Append              | 如果目标位置已经存在数据，那么将数据追加进去                 |
  | SaveMode.Overwrite           | 如果目标位置已经存在数据，那么将已经存在的数据删除，用新数据进行覆盖 |
  | SaveMode.Ignore              | 如果目标位置已经存在数据，那么就忽略，不做任何操作。         |

​	 

 - 数据源Parquet
    - Parquet时面向分析业务的列式存储格式，由Twitter和Clodera合作开发，2015年5月从Apache的孵化器里毕业成为Apache顶级项目
   - 列式存储和行式存储相比有哪些优势？
     - 1、可以跳过不符合条件的数据，只读取需要的数据，降低IO数据量。
     - 2、压缩编码可以降低磁盘存储空间。由于同一列的数据类型是一样的，可以使用更高效的压缩编码（Run Length Encoding和Delta Encoding）进一步节约存储空间。
     - 3、只读需要的列，支持向量运算，能够获取更好的扫描性能。
- JDBC数据源
  - Spark SQL支持使用JDBC从关系型数据库（比如Mysql）中读取数据。读取的数据，依然由DataFrame表示，可以很方便地使用Spark Core提供的各种算子进行处理。
  - 实际上使用Spark SQL处理JDBC中的数据是非常有用的，比如说MYSQL业务数据库中有大量的数据，比如1000万，然后需要编写一个程序对线上的脏数据进行某种负责业务逻辑处理，甚至复杂到可能设计到要用Spark SQL反复拆线呢Hive中的数据，来进行关联处理。
  - 用Spark SQL来通过JDBC数据源，加载MySQL中的数据，然后通过各种算子进行处理，是最好的选择。因为Spark是分布式的计算框架，对于1000万的数据，肯定是分布式处理的。如果自己手工编写Java程序分批次处理，耗时很长。





## 函数详解【内置函数+开窗函数】

- 内置函数

  | 种类           | 函数                                                         |
  | -------------- | ------------------------------------------------------------ |
  | 聚合函数       | approxCountDistinct, avg, count, countDistinct, first, last, max, mean, min, sum, sumDistinct |
  | 集合函数       | array_contains, explode, size, sort_array                    |
  | 日期/时间函数1 | 日期时间转换
unix_timestamp, from_unixtime, to_date, quarter, day, dayofyear, weekofyear, from_utc_timestamp, to_utc_timestamp
从日期时间中提取字段
year, month, dayofmonth, hour, minute, second |
  | 日期/时间函数2 | **日期/时间计算
datediff, date_add, date_sub, add_months, last_day, next_day, months_between
获取当前时间等
current_date, current_timestamp, trunc, date_format** |
  | 数学函数       | abs, acros, asin, atan, atan2, bin, cbrt, ceil, conv, cos, sosh, exp, expm1, factorial, floor, hex, hypot, log, log10, log1p, log2, pmod, pow, rint, round, shiftLeft, shiftRight, shiftRightUnsigned, signum, sin, sinh, sqrt, tan, tanh, toDegrees, toRadians, unhex |
  | 混合函数       | array, bitwiseNOT, callUDF, coalesce, crc32, greatest, if, inputFileName, isNaN, isnotnull, isnull, least, lit, md5, monotonicallyIncreasingId, nanvl, negate, not, rand, randn, sha, sha1, sparkPartitionId, struct, when |
  | 字符串函数     | ascii, base64, concat, concat_ws, decode, encode, format_number, format_string, get_json_object, initcap, instr, length, levenshtein, locate, lower, lpad, ltrim, printf, regexp_extract, regexp_replace, repeat, reverse, rpad, rtrim, soundex, space, split, substring, substring_index, translate, trim, unbase64, upper |
  | 窗口函数       | cumeDist, denseRank, lag, lead, ntile, percentRank, rank, rowNumber |

  

## spark sql工作原理剖析以及性能优化

 - 设置shuffle过程中的并行度：spark.sql.shuffle.partition(200)
 - 在Hive数据仓库建设过程中，合理设置数据类型，比如能设置int的就不要设置为bigint，减少数据类型导致的不必要的内存开销。
 - 编写SQL的时候，尽量给出明确的列名，比如select name from students。不宜写select * 
 - 并行处理查询结果：对于Spark SQL查询的结果，如果数据量比较大，比如超过1000条，那么就不要一次性collect()到Driver再处理。使用foreach()算子，并行处理查询结果。



## hive on spark



 - Hive是目前大数据领域，事实上的SQL标准。其底层默认是基于MapReduce实现的，但是由于MapReduce速度实在比较慢，因此这两年，陆续出来了新的SQL查询引擎。包括Spark SQL，Hive On Tez，Hive On Spark等。
 - Spark SQL与Hive On Spark是不一样的。Spark SQL是Spark自己研发出来的针对各种数据源，包括Hive、JSON、Parquet、JDBC、RDD等都可以执行查询的，一套基于Spark计算引擎的查询引擎。因此它是Spark的一个项目，只不过提供了针对Hive执行查询的工功能而已。适合在一些使用Spark技术栈的大数据应用类系统中使用。
 - 而Hive On Spark，是Hive的一个项目，它是指，不通过MapReduce作为唯一的查询引擎，而是将Spark作为底层的查询引擎。Hive On Spark，只适用于Hive。在可预见的未来，很有可能Hive默认的底层引擎就从MapReduce切换为Spark了。适合于将原有的Hive数据仓库以及数据统计分析替换为Spark引擎，作为全公司通用的大数据统计分析引擎。

## 案例-每日Top3热点搜索词统计

 - 数据格式：

    - 日期 用户 搜索词 城市 平台 版本

 - 需求：

    - 1、筛选出符合查询条件（城市、平台、版本）的数据
    - 2、统计出每天搜索uv排名前3的搜索词
    - 3、按照每天的top3搜索词的uv搜索总次数，倒序排序
    - 4、将数据保存到hdfs中

- 实现思路

  - 1、针对原始数据（HDFS文件），获取输入的RDD

  - 2、使用filter算子，去针对输入RDD中的数据，进行数据过滤，过滤出符合查询条件的数据。

    - 2.1 普通的做法：直接在fitler算子函数中，使用外部的查询条件（Map），但是，这样做的话，是不是查询条件Map，会发送到每一个task上一份副本。（性能并不好）
    - 2.2 优化后的做法：将查询条件，封装为Broadcast广播变量，在filter算子中使用Broadcast广播变量进行数据筛选。

  - 3、将数据转换为“(日期_搜索词, 用户)”格式，然后呢，对它进行分组，然后再次进行映射，对每天每个搜索词的搜索用_

    户进行去重操作，并统计去重后的数量，即为每天每个搜索词的uv。最后，获得“(日期_搜索词, uv)”

  - 4、将得到的每天每个搜索词的uv，RDD，映射为元素类型为Row的RDD，将该RDD转换为DataFrame

  - 5、将DataFrame注册为临时表，使用Spark SQL的开窗函数，来统计每天的uv数量排名前3的搜索词，以及它的搜索uv，最后获取，是一个DataFrame

  - 6、将DataFrame转换为RDD，继续操作，按照每天日期来进行分组，并进行映射，计算出每天的top3搜索词的搜索uv的总数，然后将uv总数作为key，将每天的top3搜索词以及搜索次数，拼接为一个字符串

  - 7、按照每天的top3搜索总uv，进行排序，倒序排序

  - 8、将排好序的数据，再次映射回来，变成“日期_搜索词_uv”的格式

  - 9、最后把数据保存到hdfs中

