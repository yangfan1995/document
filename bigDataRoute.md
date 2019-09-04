### Hadoop核心

#### 分布式存储基石：HDFS

HDFS简介入门演示构成及工作原理解析：数据块，NameNode,DataNode、数据写入与读取过程、数据复制、HA方案、文件类型、HDFS常用设置JavaAPI代码演示

#### 分布式计算基础：MapReduce

MapReduce简介、编程模型、JavaAPI介绍、编程案例介绍、MapReduce调优

#### Hadoop集群资源管家：YARN

YARN基本架构资源调度过程调度算法YARN上的计算框架

### 离线计算

#### 离线日志收集利器：Flume

核心组件介绍，Flume实例：日志收集、适宜场景、常见问题。

#### 离线批处理必备工具：Hive

Hive在大数据平台里的定位、总体架构、使用场景之AccessLog分析HiveDDL&DML介绍视图函数（内置，窗口，自定义函数）表的分区、分桶和抽样优化。

#### 更快更强更好用的MR：Spark

Scala&Spark简介基础Spark编程（计算模型RDD、算子Transformation和Actions的使用、使用Spark制作倒排索引）SparkSQL和DataFrame实例：使用SparkSQL统计页面PV和UV。

### 实时计算

#### 流数据集成神器：Kafka

Kafka简介构成及工作原理解析4组核心API生态圈代码演示：生产并消费行为日志。

#### 实时计算引擎：SparkStreaming

工作原理，编写Streaming程序的一般过程，如何部署Streaming程序，如何监控Streaming程序，性能调优

#### 海量数据高速存取数据库：HBase

架构及基本组件HBaseTable设计HBase基本操作访问HBase的几种方式。

### 大数据ETL

#### ETL神器：Sqoop

抽取Mysql数据到Hive实战Sqoop介绍、抽取Hive数据到Mysql实战。

#### 任务调度双星：Oozie，Azkaban

ETL与计算任务的统一管理和调度，Crontab调度的方案自研调度系统的方案开源系统Oozie和Azkaban，Azkaban的定时调度，任务分片,任务提交

#### 数据同步 DataX

DataX 是一个异构数据源离线同步工具,致力于实现包括关系型数据库(MySQL、Oracle等)、HDFS、Hive、ODPS、HBase、FTP等各种异构数据源之间稳定高效的数据同步功能。部署实践，数据导入导出

#### 数据建模 OneData

OneData是阿里巴巴内部进行数据整合及管理的方法体系和工具。其他建模：实体关系（ER）模型,维度模型,DataVault，Anchor模型，具体方式：数据规范定义，数据分为ODS（操作数据）层、CDM（公共维度模型）层、ADS（应用数据）层
