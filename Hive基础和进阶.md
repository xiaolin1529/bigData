# Hive基础和进阶

# Hive简介

 - **Hive是基于hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，提供类SQL查询功能**
    - hive中包含一个可以把sql语句转换为mapreduce的任务引擎
    - hive中不存储数据，数据是存储在hdfs中
    - hive的元数据（hive表的数据结构，表名，字段名等，表大概有30~40张）存储在mysql中。
- 使用Hive原因
- Hive特点
- Hive体系架构
- 基本命令set
- Hive日志

# Hive数据库操作

 - 查看数据库
 - 选择数据库
 - 创建数据库
 - 删除数据库

# Hive 表操作

 - 创建表
 - 查看表
 - 查看表结构
 - 查看表的创建信息
 - 表在hdfs位置
 - 重命名表
 - 加载数据到表的两种方式
 - 给表添加一个字段
 - 给表添加注释
 - 删除表

# 数据加载的两种模式

 - 读模式
 - 写模式

# Hive的基本数据类型

 - 基本数据类型
 - 复杂数据类型

# Hive表类型

 - 内部表
 - 外部表
 - 分区表
 - 桶表
 - 视图
 - 索引



# Hive存储格式

 - textfile
 - sequencefile
 - rcfile
 - orcfile
 - PARQUET

# Hive高级函数

 - 基本聚合函数
 - 扩展函数
 - 性能比较
 - 自定义函数

# Hive性能优化

 - 中间压缩
 - 数据倾斜

# Hive常用面试题

 - Hive内部表和外部表区别
 - Hive数据倾斜和调优
 - Hive文件压缩格式？压缩效率？
 - Hive分组排序，组内TopN
 - Hive的行转列，列转行
 - Hive如何实现UDF
 - Hive的元数据存放位置
 - Hive有哪几种表类型
 - distinct by和group by区别
 - sortby和order by区别





