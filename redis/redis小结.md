notify-keyspace-event
AKE,AK,AE,K$,EL

# redis
save:阻塞式
bgsave:创建子进程
>rdb是在数据库启动时候载入的

AOF更新的频繁，如果存在AOF，则使用AOF进行数据库初始化，如果AOF关闭则使用rdb进行初始化。

载入函数
rdb.h/int rdbLoad(char *filename);

为了防止多线程竞争，bgsave执行的时候禁止save，bgsave,拒绝save命令

为了减小磁盘操作
bgsave先，延迟bgrewriteaof到bgsave执行结束
bgrewriteaof先，拒绝bgsave

client-output-buffer-limit
client-output-buffer-limit 名称 硬性限制 软性限制 限制判断时长

redis 慢查询
slowlog-log-slower-than 10000 单位微妙
slowlog-max-len 128 最大条数，FIFO队列

serverCron函数 定时循环 100ms执行一次
用于精度不高操作上，高精度需要调用系统时间
1. 调用clientCron 长时间没有活动，或者超出输出缓冲区，关闭客户端
2. 调用databaseCron 删除过期键，收缩字典

lru clock 用于计算空转时长的缓存时间

SYNC耗时耗资源

复制积压缓冲区 repl-backlog-szie
second(s) * write_size_per_second

心跳检测
补发缺失数据 replconf ack

Sentinel是根据INFO命令发现从服务器

down-agter-millisecond(ms) 主观下线时间->SRI_S_DOWN
is-master-down-by-addr 询问是否同意下线


sentinel 故障转移 判断数据完整？->偏移量最大的？
从服务器优先权？->配置slave-priority 100
