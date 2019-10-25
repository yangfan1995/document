# Hive压缩、文件配置优化

## 背景

### 文件压缩的意义

文件压缩优点体现两个方面

1. 减少文件的物理存储空间，数据大的情况下适当的压缩方式可以减少很多存储空间。
2. 减小文件在网络上的网络传输开销，减少传输时间。在大数据的MapReduce条件下，job间通信频繁，网络传输频繁，对中间数据结果进行压缩，可以缩短大量的时间。

一般MR(MapReduce)操作都是IO密集的，也就是磁盘的读写比较频繁，相对来说CPU的使用率不是特别高，而压缩刚好是CPU密集操作。

### 行式存储和列式存储

传统型数据库是采用行存储形式，所有的数据都会放在一行上，便于进行更新添加操作删除等操作。最小的操作单元是行，所以想要读取某一列的数据时也会将所有数据读取出来。所有的数据在磁盘上线性存储的。

| id   | user_id | user_name |
| ---- | ------- | --------- |
| 1    | 0001    | 张三      |
| 2    | 0002    | 李四      |
| 3    | 0003    | 王五      |

*行存储*磁盘上的存储为`1,0001,张三`->`2,0002,李四`->`3,0003,王五`

*列存储*磁盘上的存储为`1,2,3`->`0001,0002,0003`->`张三,李四,王五`

#### 嵌套数据模型

Hive是支持Array(数组)，Map(键值对)，Struct(结构体)等复杂结构的数据类型，如果保存的数据中是复杂数据结构的嵌套，那么就要考虑存储和压缩的性能问题。

## 压缩算法

| 压缩格式 | codec类     | 算法    | 扩展名  | 工具  | hadoop自带 | 压缩速度 | 压缩比率 |
| -------- | ----------- | ------- | ------- | ----- | ---------- | -------- | -------- |
| gzip     | GzipCodec   | deflate | .gz     | gzip  | 是         | 中       | 中       |
| bzip2    | Bzip2Codec  | bzip2   | .bz2    | bzip2 | 是         | 慢       | 小       |
| lzo      | LzopCodec   | lzo     | .lzo    | lzop  | 否         | 快       | 大       |
| snappy   | SnappyCodec | snappy  | .snappy | 无    | 否         | 快       | 大       |

## 常用文件格式

| 存储类型 | 是否可分 | 是否默认压缩 | 压缩格式 | 支持的压缩格式   | 存储结构 |
| -------- | -------- | ------------ | -------- | ---------------- | -------- |
| TextFile | 是       | 否           |          | Gzip             | 行存储   |
| ORCFile  | 是       | 是           | zlib     | zlib,snappy,none | 行列存储 |
| Parquet  | 是       | 是           | Snappy   | Snappy、Gzip     | 列存储   |

### Orc格式

#### 文件格式和检索方式

Orc是常见的OLAP文件格式，支持复杂的数据类型，行列式存储方式。将一个文件分为三个等级，File Level，Stripe Level，Row-Group Level。每层都会有相应的索引信息来加速查询。

![image-20191024102057756](https://s2.ax1x.com/2019/10/24/KN1HDx.png)

Stripe相当于关系型数据库的页的定义，为了增加每次IO读取的数据量，加快检索，大小为250M。每10000行数据为一个Row-Group。

| key                  | Default   | Note               |
| -------------------- | --------- | ------------------ |
| orc.compress         | ZLIB      | 默认压缩算法       |
| orc.compress.size    | 262,144   | 每个压缩数据块大小 |
| orc.stripe.size      | 268435456 | stripe的大小       |
| orc.row.index.stride | 10,000    | 数据步长           |
| orc.create.index     | true      | 是否创建索引       |

#### 简单事务

Orc支持简单的事务包含删除，更新操作。对每一个事务按照顺序生成一个的delta文件，记录了事务的操作，当文件个数足够多的，将所有的事务操作进行合并，并在原始数据上合并执行。

### Parquet格式

 Apache Parquet是Apache Hadoop生态系统的一种免费的开源面向列的数据存储格式，也是一个项目，包含了很多子模块，比如和Rust交互，C++交互，文件格式化为Parquet文件。

#### 文件格式

每一个数据模型的schema包含多个字段，每一个字段又可以包含多个字段，每一个字段有三个属性：重复数、数据类型和字段名。重复数可以是以下三种：required(出现1次)，repeated(出现0次或多次)，optional(出现0次或1次)。每一个字段的数据类型可以分成两种：group(复杂类型)和primitive(基本类型)。

 ![Parquet官网示例](https://s2.ax1x.com/2019/10/24/KNNvwR.png)
 ![节点示例图](https://s2.ax1x.com/2019/10/25/Kd95sH.png)

1. Repetition Level：重复级别，在写入的时候该值等于它和前面的值在哪一层节点是不共享的。
2. Definition Level：仅仅对于空值是有效的，指明该列的路径上多少个可选字段被定义了。如果一个字段是定义的，那么它的所有的父节点都是被定义的，如果字段重复数为required，不计算level。计算方式：从根节点开始遍历，如果一个字段有值，当前值的Definition Level是当前值所在的level（实际level-该路径上Required节点个数），如果字段没有值，该字段的路径上的节点开始是空的时候的深度作为当前值的Definition Level。

## 压缩优化

从背景两个方面区分，压缩主要是针对原始数据压缩和计算结果压缩。

### 原始数据压缩

通过文件格式的结构说明和当前具体场景，业务一般采用Parquet文件或Orc文件的方式进行存储数据，在建立表结构时候进行压缩方式的定义。

### 中间job通信结果压缩

通过压缩数据，减小Job间网络通信的开销，可通过一下参数配置

```bash
set hive.exec.compress.intermediate=true
set mapred.map.output.compression.codec=org.apache.hadoop.io.compress.DefaultCodec
set mapred.map.output.compression.type=(BLOCK/RECORD) 
```

### reduce结果压缩

最终的reduce输出数据结果的压缩

```bash
set hive.exec.compress.output=true
set mapred.output.compression.codec=org.apache.hadoop.io.compress.DefaultCodec
set mapred.output.compression.type=(BLOCK/RECORD) 
```

## 优化总结参考

1. 文件格式首选Orc文件格式，指定压缩格式，如果数据结构比较复杂，选用Parquet文件，原始数据表，选用gzip，bzip2压缩方式，中间经常读取的数据表使用Snappy压缩方式
2. 开启中间结果压缩，一般选用Snappy压缩方式，压缩快，解压快
3. 开启reduce结果压缩，一般采用Snappy压缩，如果数据不再进行处理，归档选用bzip2压缩方式，压缩比率更高，占用空间更小





参考文章：

[MySql页定义]( https://segmentfault.com/a/1190000008545713 "MySql中页定义")

[MySql中MVCC事务控制]( https://draveness.me/database-concurrency-control )draveness.me/database-concurrency-control )