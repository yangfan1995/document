# 键值对操作

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

java 可通过InputFormat或者OutputFormat 访问数据，不同的文件系统会有不同的配置和压缩方式

## 文件格式

| 文件格式        | 结构化   | 备注             |
| --------------- | -------- | ---------------- |
| 文本文件        | 否       |                  |
| json            | 半结构化 |                  |
| csv             | 结构化   |                  |
| sequenceFiles   | 结构化   |                  |
| Protocal buffer | 结构化   |                  |
| 对象文件        | 结构化   | 依赖于Java序列化 |

### 文本文件

#### 读取

可以一行为一个RDD元素，或者多个文本作为Pair RDD

```scala
var input = sc.textFile(path)
```

针对许多数据可以使用wholeTextFile()生成多文件的Pair RDD，文件路径支持模糊匹配的方式。

```scala
var input = sc.wholeTextFile(path)
```

#### 保存

如果是PairRDD，会保存多个文本文件。

```scala
result.saveAsTextFile(path)
```

### JSON

#### 读取

```scala
case class Person(name: String, lovesPandas: Boolean)
val result = input.flatMap(record => {
try {
	Some(mapper.readValue(record, classOf[Person]))
} catch {
	case e: Exception => None
}})
```

#### 保存

```scala
result.map(mapper.writeValueAsString(_)).saveAsTextFile(path)
```

### CSV

#### 读取

 `Hadoop InputFormat`不支持包含换行符的记录

```scala
val input = sc.textFile(inputFile)
val result = input.map{ line =>
    val reader = new CSVReader(new StringReader(line));
    reader.readNext();
}
```

*文件解析如果文件过大则会导致解析时候成为性能瓶颈*

#### 保存

```scala
pandaLovers.map(person => List(person.name, person.idCard).toArray)
.mapPartitions{
    people =>
        val stringWriter = new StringWriter();
        val csvWriter = new CSVWriter(stringWriter);
        csvWriter.writeAll(people.toList)
        Iterator(stringWriter.toString)
}.saveAsTextFile(outFile)
```

### SequenceFile

#### 读取

`sequence`文件是通过参数`Writable`确定具体的类型

```scala
var data = sc.sequenceFile(path,keyClass,valueClass,minPartitions)
/*val data = sc.sequenceFile(inFile, classOf[Text], classOf[IntWritable]).map{case (x, y) => (x.toString, y.get())}*/
```

#### 保存

暂无

### 对象文件

对象文件的局限性，是如果对象字段修改需要，保存的对象文件不再可读

#### 读取

```scala
var data = sc.hadoopFile()
```

#### 保存

