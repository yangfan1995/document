# Spark工作机制

## 调度管理

### 名词解释

1. driver 程序：用户编写的Spark程序，一般在worker节点上执行
2. SparkContext对象：负责与集群进行沟通
   1. 联系集群管理器，进行资源分配
   2. 启动专属本程序的工作节点上的执行器
   3. 程序代码分发到工作节点
   4. SparkContext分发task到各执行器
3. 集群管理器

### Spark 内部程序调度

#### 资源调度

 	1. 静态分配
     + Standalone模式
     + Mesos
     + YARN
 	2. 动态分配
 	3. 移除策略

### Spark程序内部调度

> 默认情况下是FIFO调度，改为循环方式，适用于多用户场景，spark.scheduler.mode为FAIR

1. 公平调度池
2. 调度池的默认行为
3. 调度池配置
   - mode:FIFO/FAIR
   - weight:权重
   - minShare:最小数量

## 内存管理

### RDD持久化

persist()或者cache()方法，persist()可以指定存储级别

### 共享变量

1. 广播变量
2. 计数器

## 容错机制

### 容错体系

+ 宽依赖：父RDD对应多个子RDD
+ 窄依赖：父RDD对应一个子RDD

### Master 节点失效

1. Standalone模式：依赖于Zookeeper的选举机制
2. 单点模式：通过记录错误文件，重启恢复

### Slave节点失效

1. driver：使用检查点进行重启，恢复数据块，继续重新计算RDD
2. worker：会将启动器停止
3. 执行器：定时接收StatusUpdate,没有收到会将执行器移除，worker收到指令后才会重新启动执行器

# Spark内核讲解

## Spark核心数据结构RDD

###RDD:弹性分布式数据集

1. 数据集：数据为平铺，可以遍历
2. 分布式：分布式存储，数据被分割为多个水平数据块，分布在多个节点上，便于并行计算
3. 弹性：一些操作可以在数据块上直接计算，比如map()，有些数据是要所有数据块同时操作，比如groupByKey

RDD特点：

1. 可重复计算：容错性，数据缺失后可以重新计算得出结果
2. 只读性：数据不可更改，为了简化操作，不用考虑数据互斥
3. 可缓存：RDD生成一次后会被jvm回收，多次使用多次计算，如果为了计算更快，可进行缓存

### RDD定义

#### 分区和依赖

```scala
  // Our dependencies and partitions will be gotten by calling subclass's methods below, and will
  // be overwritten when we're checkpointed
  private var dependencies_ : Seq[Dependency[_]] = _
  @transient private var partitions_ : Array[Partition] = _
```

#### 计算函数

```scala
/** * :: DeveloperApi :: * Implemented by subclasses to compute a given partition. */
@DeveloperApidef compute(split: Partition, context: TaskContext): Iterator[T]
```

#### 分区

```scala
/** Optionally overridden by subclasses to specify how they are partitioned. */
@transient val partitioner: Option[Partitioner] = None
```

### RDD的Transformation

RDD通过构造有向无环图完成容错的机制，计算更加便捷，比如map，filter，如果链条过长，需要通过检查点checkPoint机制，保存快照进行容错。

### RDD的Action

RDD的action调用后不再生成新的RDD，返回Driver

### Shuffle

数据混洗：成本很高，主要是网络传输的开销

<img src="E:\image\img\reduceByKey.png" alt="reduceByKey" style="zoom:50%;" />

## SparkContext

SparkContext.createTaskScheduler分别针对不同调度方式区分主调度器和备调度器

```scala
master match {
        //主调度器
    case "local" =>
    val scheduler = new TaskSchedulerImpl(sc, MAX_LOCAL_TASK_FAILURES, isLocal = true)
        //备用调度器
    val backend = new LocalSchedulerBackend(sc.getConf, scheduler, 1)
    scheduler.initialize(backend)
    (backend, scheduler)
}
```

# SparkSQL和数据仓库

> SparkSQL是Spark的一个模块，支持很多数据源方式容错。

SparkSQL分为两种方式，一种是直接写SQL方式一种是通过API实现DataFrame的操作

## SparkSQL

### 支持的语法

普通的SQL语法，1.4开始后支持窗口函数

### 支持的数据类型

1. 数字类型：ByteType,ShortType,IntegerType,LongType,FloatType,DoubleType,DecimalType
2. 字符串类型：StringType
3. 二进制：BinaryType
4. 布尔类型：BooleanType
5. 日期类型：TimeStampType,DateType
6. 复杂类型：StructType,ArrayType,MapType

### DataFrame 数据源

写入模式：

1. SaveMode.ErrorIfExists
2. SaveMode.Append
3. SaveMode.Overwrite
4. SaveMode.Ignore

### SparkSQL架构

![1568810830702](E:\image\img\SparkSQL架构.png)

### Catalyst 

#### 执行步骤

##### 分析

通过语法树或者数据的DataFrame对象，查找关系表，匹配属性名，指向相同属性的赋予相同ID，扩散定义类型。

##### 逻辑优化

针对基于规则的优化方式

##### 物理优化

对于已知的小表使用广播连接，递归整个树来计算成本

##### 代码生成

# SparkStreaming

## SparkStreaming基础知识

### 基本概念

#### StreamingContext

基本环境对象，提供基本的功能入口，构建DStream

### DStream

Streaming的核心抽象，是一系列的RDD序列，每一个RDD是一个时间片周期的数据

```mermaid
graph LR
    时间0-1数据 --> 时间1-2数据
    时间1-2数据 --> 时间12-3数据
```

### DStream的输入

#### 输入类型

##### 基本类型

 >Api提供方式，文件、socket

##### 高级类型

 >Kafka、Flume

### DStream的操作

1. Transformation:类似RDD的transformation
2. output:类似RDD的action

窗口操作
> 主要是窗口长度和滑动区间确定，窗口长度代表处理长度，滑动区间代表处理间隔

Output操作
+ `print()`,测试调试使用
+ `saveAsTextFile(prefix,[suffix])`,保存为文件文本
+ `saveAsObjectFile(prefix,[suffix])`,保存为SequenceFile
+ `foreachRDD(func)`,根据传入函数作用于RDD

### 容错处理

1. 数据源容错
2. 检查点数据处理容错
3. 写出覆盖容错

### 性能处理

> 主要两个方向，批次处理事件尽量短和数据尽快处理

1. 减少批处理事件

   增加数量处理并发量，如果数据处理的并发量不是平衡需要调整，尽量使用Kyro方式来处理数据减小CPU和内存占用，task是否频繁启动

2. 合理的处理间隔

[^图，机器学习]: 暂时不去了解

