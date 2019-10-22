# Hive 分区分桶以及索引

## 背景

HDFS是作为分布式文件系统，是将文件通过切分为文件块存储在不同的服务器上，为了实现高可用，默认每个文件块都会冗余存在三份数据在不同的服务器。节点分为DataNode和NameNode，DataNode主要负责数据存储，NameNode会管理所有的块信息。Hive会将所有的任务转换为MapReduce（MR）任务，一个Task会有多个MR组成。

以下是针对存储相关的Hive中的分桶分区优化进行整理，先对Hadoop的文件块进行一部分拓展作为后续的基础，从Hive分区分桶的场景和优化方式以及原理介绍相关内容，因为Hive索引的缺陷在Hive3.0废弃，没有对索引内容进行展开。

## HDFS 文件系统

HDFS中文件被拆成多个块，并且集中存储在DataNode多个节点上，Hadoop2.X默认文件分块大小为128M，`NameNode`上通常存储集群中文件块的存储位置。Hadoop提供了一个抽象的文件系统，大体可以分为两类一部分文件的读写操作，一部分进行文件目录的事务，在`org.apache.hadoop.fs.FileSystem`中定义,所有的文件状态都保存在`FileStatue`类中，副本个数和块个数等HDFS等特殊参数。

### 分块大小确定

- 减少硬盘寻道时间

- 减少NameNode内存消耗

- map崩溃问题

- 监管时间问题

- 问题分解问题

- 约束map输出

  [参考文章](  https://www.cnblogs.com/Dhouse/p/6901028.html  )

### 块大小计算

1. HDFS中平均寻址时间大概为`10ms`；
2. 经前人大量测试发现，寻址时间为传输时间的`1%`是最佳状态，得出最佳传输时间=`10ms / 1%`=`1000ms`=`1s`；
3. 目前磁盘的传输速率普遍为`100MB/s`
4.  block size = `100MB/s * 1s`= `100MB` 

## Hive分区和分桶

> 分区针对是针对数据存储路径，分桶是针对数据文件的存储。

### Hive分区

#### 场景

Hive中每个分区对应着表很多的分区文件夹，将所有的数据按照分区列放入到不同的分区文件夹中去。对表的分区的划分应该让数据尽可能均匀分布。

![Hive分区表](https://s2.ax1x.com/2019/10/22/K8x2Is.png)

![分区文件夹](https://s2.ax1x.com/2019/10/22/KGSq81.png)

最好的情况下，分区的划分条件总是能够对应where语句的部分查询条件。当where条件谓词查询分区列时，只需要扫描分区列目录下的数据，不扫描其他不关心的分区，快速定位文件夹。

分区方式主要分为动态分区和静态分区。静态分区指在创建表时，进行分区规划；动态分区则是具体的处理数据进行分区目录的创建。

#### 优化方式

##### 动态分区方式

1. 开启动态分区`set hive.exec.dynamic.partition=true`
2. `set hive.exec.dynamic.partition.mode=nonstrict`*动态分区的模式，默认strict，表示必须指定至少一个分区为静态分区，`nonstrict`模式表示允许所有的分区字段都可以使用动态分区*
3. ` set hive.exec.max.dynamic.partitions=1000 `*分区个数过多指定在所有节点上一共可以创建多少个分区*
4. ` set hive.exec.max.dynamic.partitions.pernode=100 `*每个节点最大的分区个数*
5. `set hive.exec.max.created.files=100000`*指定每个job可以创建多少个HDFS文件*

> 分区提供了一个隔离数据和优化查询的可行方案，但是并非所有的数据集都可以形成合理的分区，分区的数量也不是越多越好，过多的分区条件可能会导致很多分区上没有数据。同时Hive会限制动态分区可以创建的最大分区数，用来避免过多分区文件对文件系统产生负担。 

### Hive分桶

#### 场景

1. 桶是相对分区更为细粒度的数据划分，使得效率查询效率更高；
2. 加快表连接（小表join大表）的速度；
3. 数据抽样

> 其他情况不建议分桶，分桶表是一经决定，就不能更改，所以如果要改变桶数，要重新插入分桶数据。

#### 原理

分桶表是对列值取**哈希值**的方式，将**不同数据**放到**不同文件**中存储。

#### 优化方法

+ 开启分桶` set hive.enforce.bucketing=true; `*Hive2.0不需要设置*
+  `set hive.optimize.bucketmapjoin=true `*改变默认join行为*
+  `set hive.optimize.bucketmapjoin.sortedmerge = true; `*如果两个表中分区字段是有序的*

> load加载数据不会进行分桶操作，使用CTAS方式才会进行分桶操作

## Hive索引机制

### Hive索引

#### 场景

索引的设计目标是提高表某些列的查询速度。如果没有索引，带有谓词的查询会加载整个表或分区并处理所有行。但是如果 column 存在索引，则只需要加载和处理文件的一部分。

#### 原理

##### 压缩索引

针对建立索引的列会创建如下索引表，

| col_name    | data_type     | comment       |
| ----------- | ------------- | ------------- |
| col_name    | col_type      | 列名          |
| _bucketname | string        | HDFS 文件路径 |
| _offset     | array<bigint> | 偏移量        |

#### 优化方式

+ 默认情况下hive查询是不会通过索引的，可以通过以下配置方式进行开启

    1. `hive.optimize.index.filter;`*是否开启索引查询*
    2. `hive.optimize.index.filter.compact.minsize;`*输入大于最小的阀值会自动应用索引*
    3. `hive.optimize.index.filter.compact.maxsize;`*使用压缩索引查询时能读到的最大索引项数 ，负值表示无穷*
    4. `hive.exec.concatenate.check.index`*删除表或者分区时是否检查索引，如果有索引会报错提醒*
    5. `hive.index.compact.binary.search`*是否启用二分查找*
+ 适用于不更新的静态字段
+ 不要建立过多的索引

#### 缺陷

1. 不够智能化，每次建立更新数据都要重新构建索引
2. hive索引表是物理表，需要单独的job进行索引表的查询
3. Hive3.0删除适用物化视图和Parquet文件或ORC文件代替，两种文件方式都会实现轻量的索引

## 优化总结

1. 根据具体文件大小去修改HDFS块大小，默认为128M
2. 对于数据量较大的集合进行分区操作，如果是动态分区通过参数规划好分区个数和文件个数
3. 如果分区内数据依旧过大，对分区内数据进行分桶操作，尽量使用` DISTRIBUTE BY id SORT BY id `进行排序，优化查询和join时的操作，修改分桶条件后，数据需要重新导入
4. 文件尽量使用ORC文件和Parquet文件存储，并启用索引检索配置，建表时启用ORC文件默认轻量索引优化查询并对数据进行sorted by排序，推荐ORC文件方式。