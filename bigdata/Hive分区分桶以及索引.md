# Hive 分区分桶以及索引

## HDFS 文件系统
HDFS中文件被拆成多个块，并且集中存储在DataNode多个节点上，Hadoop2.X默认文件分块大小为128M，`NameNode`上通常存储集群中文件块的存储位置。Hadoop提供了一个抽象的文件系统，大体可以分为两类一部分文件的读写操作，一部分进行文件目录的事务，在`org.apache.hadoop.fs.FileSystem`中定义,所有的文件状态都保存在`FileStatue`类中，副本个数和块个数等HDFS等特殊参数。
### 分块大小确定
+ 减少硬盘寻址时间
+ 减少Namenode内存消耗
+ map崩溃问题
+ 监管时间问题
+ 问题分解问题
+ 约束map输出

## Hive分区和分桶

> 分区针对是针对数据存储路径，分桶是针对数据文件的存储。

### Hive分区

#### 场景

Hive中每个分区对应着表很多的子目录，将所有的数据按照分区列放入到不同的子目录中去。对不同列的划分应该让数据尽可能均匀分布。最好的情况下，分区的划分条件总是能够对应where语句的部分查询条件。当where条件给出列值，只需要扫描目录下的数据，不扫描其他不关心的分区，快速定位文件夹，主要分为动态分区和静态分区。静态分区指在创建表时，进行分区规划；动态分区则是具体的处理数据进行分区目录的创建。

#### 原理



#### 优化方式

##### 静态分区方式

+ 建表时指定分区字段和字段类型

   ```sql
   create table tmp_table (name string,address string) partitioned by (sex string) row format delimited fields terminated by ',';
   ```

+ 修改表结构添加分区字段和字段类型

   ```sql
   alter table tmp_table add if not exists partition(sex='male') 
   ```

##### 动态分区方式

1. 开启动态分区`set hive.exec.dynamic.partition=true`
2. 如果是有静态分区存在，也就是静态和动态分区都存在，需要指定`set hive.exec.dynamic.partition.mode=nonstrict`
3. 如果分区个数过多可以通过` set hive.exec.max.dynamic.partitions=1000 `指定在所有节点上一共可以创建多少个分区；` set hive.exec.max.dynamic.partitions.pernode=100 `,每个节点最大的分区个数
4. `set hive.exec.max.created.files=100000`指定每个job可以创建多少个HDFS文件

### Hive分桶

#### 场景

1. 桶是相对分区更为细粒度的数据划分，使得效率查询效率更高；
2. 加快表连接map-join的速度；
3. 数据抽样

> 其他情况不建议分桶，分桶表是一经决定，就不能更改，所以如果要改变桶数，要重新插入分桶数据。

#### 原理

1. 分桶表是对列值取**哈希值**的方式，将**不同数据**放到**不同文件**中存储。
2.  对于hive中每一个表、分区都可以进一步进行分桶 

#### 优化方法

+ 开启分桶` set hive.enforce.bucketing=true; `
+ 

## Hive索引机制

### Hive索引

#### 场景

#### 原理

#### 优化方式



