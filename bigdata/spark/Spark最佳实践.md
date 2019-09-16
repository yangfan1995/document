# Spark工作机制

## 调度管理

+ 静态分配

+ 动态分配

+ 移除策略

### Spark 内部程序调度

> 默认情况下是FIFO调度，改为循环方式，适用于多用户场景，spark.scheduler.mode为FAIR

1. 公平调度池
2. 调度池的默认行为
3. 调度池配置
   + mode:FIFO/FAIR
   + weight:权重
   + minShare:最小数量