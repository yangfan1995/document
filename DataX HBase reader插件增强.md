# DataX HBase reader插件增强

## 业务场景

业务需要通过DataX进行数据的拉取操作，现阶段DataX仅支持range中的startKey和endKey方式进行范围过滤操作，业务需要通过`minTimestamp`和`maxTimestamp`方式进行范围查询。

## 修改步骤

### 配置文件修改

1. `src/main/resource/plugin.json`指定插件的入口方式
2. `src/main/resource/plugin_job_template.json`指定json配置文件的模板类型

### 代码文件修改

1. 常量文件修改

   `com.alibaba.datax.plugin.reader.hbase11xreader.Key`文件添加时间戳配置字段`minTimestamp`和`maxTimestamp`，为了减少改动量，`com.alibaba.datax.plugin.reader.hbase11xreader.Constant`中保持原配置文件结构不变，`minTimestamp`和`maxTimestamp`依旧包含于`range`属性下，如果需要改动配置文件结构，需要同步修改`com.alibaba.datax.plugin.reader.hbase11xreader.Hbase11xHelper#validateParameter`中对配置文件的结构预处理的方法

2. 添加新增配置信息读取方法

   1. 新增`public static byte[] convertInnerMinTimeTimestamp(Configuration configuration)`和`public static byte[] convertInnerMaxTimestamp(Configuration configuration)`，
   2.  `com.alibaba.datax.plugin.reader.hbase11xreader.HbaseAbstractTask`增加配置文件信息读取为本地变量
   3. `com.alibaba.datax.plugin.reader.hbase11xreader.HbaseAbstractTask#prepare`通过java HBase Api`setTimeRange(long minStamp, long maxStamp)`方式设置`Scan`对象属性。
   4. 修改`com.alibaba.datax.plugin.reader.hbase11xreader.Hbase11xHelper#split`添加对于timestamp的判断分区
   5. 相关代码改动中打印日志和提示信息的修改

## 插件打包

使用`maven assembly`插件进行打包，指定输出文件夹，打包名称，使用`jar`包路径

[DataX插件开开发](https://github.com/alibaba/DataX/blob/master/hbase11xreader/doc/hbase11xreader.md)