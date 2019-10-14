Hadoop:Common和HDFS









> 核心：Common、HDFS、MapReduce



1. Common

2. Avro：数据密集型
3. HBase
4. Zookeeper
5. MapReduce
6. Hive
7. Pig
8. Mahout
9. X-RIME
10. Flume：日志收集系统
11. Sqoop：SQL-to-Hadoop，MapReduce并行化
12. Oozie



# Hadoop配置

## Hadoop Configration

### 成员变量

1. quietmode：加载配置时是否打印日志，true不打印
2. resources：通过addResource方法添加的资源
3. properties：配置后的键值对
4. overlay：应用设置的配置属性
5. finalParameters：配置final的键值对
6. classLoader：类加载器变量

### 资源加载

> xinclude机制： 用于合并`XML文档`的通用机制，通过在“主”文档中编写包含标记来自动包含其他文档或其他部分。生成的文档成为单个复合`XML信息集`。`XInclude`机制可用于合并`XML文件`或`非XML文本文件`中的内容，是xml标记语言中包含其他文件的方式 。

1. dom解析工厂
2. 忽略注释
3. 启用xinclude机制
4. 生成DocumentBuilder
5. 根据资源加载途径分别处理
   1. URL
   2. classpath
   3. InputStream
   4. Hadoop Path
6. 处理节点信息

![Configuration时序图](E:\document\img\Configuration时序图.png)

> 最新的hadoop采用StAX解析xml的方式，对比于SAX，是拉取模式，部分事件解析失败，不会影响剩余的事件，SAX方式中如果部分事件解析失败，其他事件不会发生。 



# 序列化与压缩

Hadoop序列化和java序列化的区别

java反序列化时会不停创建对象，Hadoop反序列化时对象是重用的。



## 序列化

### ObjectWritable

针对Sequence文件中某个字段的类型不统一，可以设置为ObjectWritable。

### Hadoop序列化框架

> 位于：org.apache.hadoop.io.serializer

1. Avro
2. Thrift
3. Google Protocol Buffer

## 压缩

> 应用场景：加快数据网络上传输，文件存储，Map阶段到Reduce阶段的数据交换

### 压缩简介 

Hadoop压缩考量

1. 压缩速度
2. 可分割性

> 编码解码、压缩解压缩，压缩流解压流

![1571045738394](E:\document\img\hadoop支持压缩格式.png)



![1571045778808](E:\document\img\hadoop编码解码器.png)

#### CompressionFactory

> 编码/解码器工厂类，获取当前所有的编码/解码器

#### 编码/解码步骤

1. 获取文件输入流
2. 获取当前对应的编码类型
3. 通过`CompressionCodecFactory#getCodecByClassName`获取编码器
4. 通过对应的编码器对获取的文件输出流进行编码
5. 关闭文件

#### 压缩/解压缩步骤

1. 获取文件输入流
2. 合理检查流数据是否结束
3. 循环判断数据流是否有数据
4. 进行setInput
5. 判断压缩缓存是否满
6. 压缩缓存已满，调用压缩`compress`进行压缩
7. 写入文件

#### 压缩流/解压缩流

压缩和解压缩的通用实现

![hadoop压缩流程](E:\document\img\hadoop压缩流程.png)

##





[计算密集，数据密集，IO密集]( https://blog.csdn.net/scrat_kong/article/details/84947414 )