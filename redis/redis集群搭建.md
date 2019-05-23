
# redis 集群搭建以及监测环境
按照官网的指导，要实现3主3从的集群 虚拟机单机ip:192.168.40.128


## 集群基本搭建
### 简单下载
+ 通过 `wget http://download.redis.io/releases/redis-4.0.10.tar.gz`
+ 解压缩 `tar zxvf redis-4.0.10.tar.gz`
+ 指定安装路径，切换root用户执行`make && make PREFIX=/usr/local/redis install`，可能出现权限不够的问题，sudo同样会报错，直接使用root进行操作。

***
### 安装编译工具
+ `sudo apt-get update`
+ `sudo apt-get install gcc`
+ `sudo apt-get install make`
+ `sudo apt-get install tcl`

***
### 创建redis集群文件夹
+ 因为是/usr，所以始终都是在root权限下进行操作
+ `cd /usr/local/redis`
+ `mkdir cluster`
+ `cd cluster`
+ `mkdir 7000 7001 7002 7003 7004 7005`

***
### 修改配置文件
复制redis conf内的config文件复制到六个文件夹中，并且修改以下内容
```
# 端口号  
port 7000  
# 后台启动  
daemonize yes  
# 开启集群  
cluster-enabled yes  
#集群节点配置文件  
cluster-config-file nodes-7000.conf  
# 集群连接超时时间  
cluster-node-timeout 5000  
# 进程pid的文件位置  
pidfile /home/ubuntu/redis-4.0.10/pid/redis-7000.pid
#工作文件夹
dir "/home/ubuntu/redis-4.0.10/working"
# 开启aof  
appendonly yes  
# aof文件路径  
appendfilename "appendonly-7005.aof"  
# rdb文件路径  
dbfilename dump-7000.rdb  
```
redis 的配置文件中的bind指定的是redis服务器的网卡ip，也就是redis服务器的ip

***
### 启动脚本
+ `cd /home/ubuntu/redis-4.0.10/`
+ `touch start.link.sh`为了操作简单,创建脚本
+ 修改启动脚本，为
```
#!/bin/bash
export BASE_FLOD="/usr/local/redis"
{BASE_FLOD}/bin/redis-server /usr/local/redis/cluster/7000/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/cluster/7001/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/cluster/7002/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/cluster/7003/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/cluster/7004/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/cluster/7005/redis.conf
#cd src
#./redis-trib.rb create --replicas 1 192.168.40.128:7000 192.168.40.128:7001 192.168.40.128:7002 192.168.40.128:7003 192.168.40.128:7004 192.168.40.128:7005
```
其中注释的是为了简化初始启动的，ip需要跟每个节点配置的redis.conf 中bind 属性绑定的一致
+ 启动后可以通过ps -ef | grep redis命令查询对应的线程是否启动

***
### 集群启动
+ 关联程序使用的ruby写的，所以要搭建rudy的运行环境，需要安装rudbygem
+ `sudo apt-get install ruby rubygems -y`
+ gem install redis,运行到这里会感觉十分慢，需要耐心等待，在redis安装目录下，src文件夹redis-trib.rb
+ 运行`redis-trib.rb create --replicas 1 192.168.40.128:7000 192.168.40.128:7001 192.168.40.128:7002 192.168.40.128:7003 192.168.40.128:7004 192.168.40.128:7005`,
检查配置的信息是否有错误，没有直接yes就可以.  ```[OK] All 16384 slots covered.```代表接群启动成功。

***
### 节点查看，重启
查看集群运行状态：使用命令`./redis-trib.rb check 192.168.40.128:7000`，进行集群的状态检查

***
### 性能测试
#### 自带测试工具redis-benchmark
+ `redis-benchmark -h 192.168.40.128 -p 6379 -c 100 -n 100000`
100个并发连接，100000个请求，检测 host 为 localhost 端口为6379的 redis 服务器性能。
+ `redis-benchmark -h 192.168.40.128 -p 6379 -q -d 100`
测试存取大小为100字节的数据包的性能。
+ `redis-benchmark -t set,lpush -n 100000 -q`
只测试某些操作的性能。
+ `redis-benchmark -n 100000 -q script load "redis.call(‘set’,’foo’,’bar’)"`
只测试某些数值存取的性能。

***
### 集群密码设置
集群搭建初始不需要密码，启动完成后，先看每个节点的配置文件是否有读写权限，如果没有读写权限，需要chmod修改的读写权限，通过
```
./redis-cli -c -p port
config set masterauth password
config set requirepass password
config rewrite
```
分别连接每个节点进行设置
若要重启发现连接不上，修改启动脚本 redis-.sh 99行，配置启动脚本密码启动
```@r = Redis.new(:host => @info[:host], :port => @info[:port], :timeout => 60,:password => "yangfan@1995")```

***
### 代码测试
```
/*
 *集群连接测试
 */

@Test
public void testJedisCluster() {
    Set<HostAndPort> nodes = new LinkedHashSet<>();
    //所有主机节点ip和端口
    nodes.add(new HostAndPort("192.168.40.128", 7000));
    nodes.add(new HostAndPort("192.168.40.128", 7001));
    nodes.add(new HostAndPort("192.168.40.128", 7002));
    nodes.add(new HostAndPort("192.168.40.128", 7003));
    nodes.add(new HostAndPort("192.168.40.128", 7004));
    nodes.add(new HostAndPort("192.168.40.128", 7005));
    //没有密码
    //JedisCluster cluster = new JedisCluster(nodes);
    //添加密码调用
    JedisCluster cluster = new JedisCluster(nodes, 5000, 5000, 10, "yangfan@1995", new GenericObjectPoolConfig());
    //cluster.zadd("test_1", String.valueOf(""),"id_2");
    System.out.println(cluster.zscore("test_1", "id_1"));
    try {
        cluster.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

***
### 主从模式、哨兵、集群的关系
1. 主从模式是指定复制和持久化关系，指定了主从备份的关系
2. 哨兵：当主数据库遇到异常中断服务后，开发者可以通过手动的方式选择一个从
数据库来升格为主数据库，以使得系统能够继续提供服务。主要是为了解决主从复制
手动切换主从关系的检测工具，可以自动切换主从。
3. 使用哨兵，redis每个实例也是全量存储，每个redis存储的内容都是完整的数据，
浪费内存且有木桶效应。为了最大化利用内存，可以采用集群，就是分布式存储。
即每台redis存储不同的内容，共有16384个slot。每个redis分得一些slot，
hash_slot = crc16(key) mod 16384 找到对应slot，键是可用键，如果有{}则取{}
内的作为可用键，否则整个键是可用键集群至少需要3主3从，且每个实例使用不同
的配置文件，主从不用配置，集群会自己选。


## 监控部署
### RedisLive搭建部署
***
#### 运行环境部署

1. `git clone https://github.com/kumarnitin/RedisLive.git` 下载redislive,解压缩`unzip -o -d /home/ubuntu/ RedisLive-master.zip`
2. 进入文件夹 `pip install -r requirements.txt -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com`
指定豆瓣源，下载速度更快。
3. 进入src文件夹，复制example文件，编辑
```
{
    "RedisServers":
    [
        {
        "server": "192.168.40.128",
        "port" : 7000,
        "password" : "yangfan@1995"
        },
        //...多个监听
    ],

    "DataStoreType" : "redis",

    "RedisStatsServer": //存储的redis监听接口
    {
        "server" : "127.0.0.1",
        "port" : 6379
    },

    "SqliteStatsStore" :
    {
        "path":  "/home/ubuntu/redis-4.0.10/working/redislive.db" //进行存储的文件
    }
}

```
4.
`ubuntu@ubuntu:~/redis-4.0.10$ mkdir pid`
`ubuntu@ubuntu:~/redis-4.0.10$ mkdir log`
`ubuntu@ubuntu:~/redis-4.0.10$ mkdir working` //保存aof，rdb，node-config文件

5. RedisLive分为两部分，其中一部分为监控脚本，另一部分为web服务，所以需要分别启动。
`./redis-monitor.py --duration=120`
`./redis-live.py`

6. 访问http://192.168.40.128:8888/index.html

## Q&A
1. redis.clients.jedis.exceptions.JedisNoReachableClusterNodeException: No reachable node in cluster
redis node的redis.conf 绑定ip设置为指定的redis节点ip，启动集群时只用指定ip启动，不使用192.168.40.128
2. connect refuse
关闭防火墙
3. No module named redis
    + 查看python位置 `which python`
    + 先备份 `sudo cp /usr/bin/python /usr/bin/python_cp`
    + 删除 `sudo rm /usr/bin/python`
    + 默认设置成python3.5，创建链接 `sudo ln -s /usr/bin/python3.5 /usr/bin/python`
