# Spark代码基础和资源调优

## Spark基础

### Spark,Spark SQL,Hive On Spark,Hive On MapReduce

1. Spark:一个开源的基于内存的运算框架
2. Spark SQL:Spark SQL是应用于Spark的一个组件
3. Hive On Spark:把Spark作为Hive的一个计算引擎，将Hive的查询作为Spark的任务提交到Spark集群上进行计算
4. Hive On MapReduce:最初的Hive执行方式，每次执行写回文件系统

>区别 Spark SQL和Hive On Spark在使用结构上是基本类似的，只是SQL引擎不一样，但是计算引擎都是Spark
>
>Spark SQL不再受限于Hive，只是兼容Hive
>
>Hive on Spark是一个Hive的发展计划，不在依赖于单纯的一个引擎


#### Spark SQL

Spark的主要两个组件是SQL Context和DataFrame
Spark SQL 提供SqlContext封装了Spark的相关操作
DataFrame 分布式的数据集合
