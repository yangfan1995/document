### 键值对操作

#### pair rdd

常用二元操作

| 函数名        | 使用                       |
| ------------- | -------------------------- |
| reduceByKey   | 合并相同键值对             |
| groupByKey    | 对具有相同键值对的进行分组 |
| conbineByKey  | 合并不同类型               |
| mapValues     | 不改变键，单纯改变值       |
| flatMapValues |                            |



mapValues

```scala
val list = List("hadoop","spark","hive","spark")
val rdd = sc.parallelize(list)
val pairRdd = rdd.map(x => (x,1))
pairRdd.mapValues(_+1).collect.foreach(println)//对每个value进行+1
```

|               |                                       |
| ------------- | ------------------------------------- |
| subtractByKey | 删除相同元素                          |
| join          |                                       |
| cogroup       | 将两个 RDD 中有相同键的数据分组到一起 |

#### 数据混洗操作

repartition()：通过网络方式进行数据的混洗，新建新的分区操作集合，代价比较大

coalesce()：优化过的repartition()的操作

### 数据分组

groupByKey

### 数据连接

jion,leftOuterJoin,rightOuterJoin



## Pair RDD 行动操作

| 函数         | 描述                 | 示例           |
| ------------ | -------------------- | -------------- |
| countByKey   | 根据key进行统计个数  | rdd.countByKey |
| collectAsMap | 将结果以map形式映射  |                |
| lookup(key)  | 返回给定键的所有数值 |                |



## 数据分区

partitionBy是一次转换操作，会创建一个新的RDD,不会改变原来的rdd,partitionBy的分区个数应该是和可用核心数保持一致，会根据核心数决定并行任务数目。

### 获取分区方式

pair.partitioner获取分区，如果分区后的数据需要多次使用，一般都会用persist()或者cache()方式持久化,防止一次一次进行分区操作

| 特性 | cache | persist |
| ---- | ----- | ------- |
|      |       |         |



#### 缓存级别

```scala
object StorageLevel {
  val NONE = new StorageLevel(false, false, false, false)
  val DISK_ONLY = new StorageLevel(true, false, false, false)
  val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
  val MEMORY_ONLY = new StorageLevel(false, true, false, true)
  val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
  val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
  val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
  val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
  val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
  val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
  val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
  val OFF_HEAP = new StorageLevel(true, true, true, false, 1)
}
```

# 数据的读取与保存

## 动机

java 可通过InputFormat或者OutputFormat 访问数据



