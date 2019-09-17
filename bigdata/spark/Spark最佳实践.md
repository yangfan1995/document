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

