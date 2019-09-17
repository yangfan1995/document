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

![reduceByKey](E:\document\img\reduceByKey.png)

## SparkContext

