# Hive优化

## 表层次优化

### 表分区优化

#### 场景

数据量大，需要进行数据隔离和查询优化

#### 原理

在HDFS文件系统中，每个分区就是一个文件夹，多分区情况是根据顺序构造多层文件的树形方式。当where条件给出列值，只需要扫描目录下的数据，不扫描其他不关心的分区，快速定位文件夹，主要分为动态分区和静态分区。静态分区指在创建表时，进行分区规划；动态分区则是具体的处理数据进行分区目录的创建。

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
4. `set hive.exec.max.created.files=100000`指定每个JOB可以创建多少个HDFS文件

### 表分桶优化

#### 场景

1. 当分区数量过于庞大，可能会导致文件系统崩溃；
2. 桶是相对分区更为细粒度的数据划分，使得效率查询效率更高；
3. 加快表连接map-join的速度；
4. 数据抽样

> 其他情况不建议分桶，小文件很恐怖

#### 原理

1. 分桶表是对列值取**哈希值**的方式，将**不同数据**放到**不同文件**中存储。
2.  对于hive中每一个表、分区都可以进一步进行分桶 

#### 优化方法

+ 开启分桶

### 表压缩格式

### 表文件格式

## SQL语法和参数优化

## 整体架构优化
