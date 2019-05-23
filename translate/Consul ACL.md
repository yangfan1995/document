## ACLs

### ACL HTTP API

`/acl endPoint` 是用来管理Consul的ACL令牌和策略的,[使用ACL][1],[检查ACL复制状态][2]并且[交换规则][3].有独立的页面针对`/acl endpoints`进行令牌和策略的管理.

对于更多的如何设置ACLs信息的,请查看ACL文档.

#### 引导ACLs

这个监测是有且仅有的一次对ACL的操作,如果Consul服务器的配置中没有规定`acl.tokens.master`的配置或者集群没有初始化过.`Consul 0.9.1`版本和以上版本,需要所有的服务器都要升级到版本便于操作.

提供了一种不需要在配置文件中设置任何存在密码相关信息的机制.


[1]: https://www.consul.io/api/acl/acl.html#bootstrap-acls
[2]: https://www.consul.io/api/acl/acl.html#check-acl-replication
[3]: https://www.consul.io/api/acl/acl.html#translate-rules