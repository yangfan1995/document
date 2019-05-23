## ACLs

### ACL HTTP API

`/acl endPoint` 是用来管理Consul的ACL令牌和策略的,[使用ACL][1],[检查ACL复制状态][2]并且[交换规则][3].有独立的页面针对`/acl endpoints`进行令牌和策略的管理.

对于更多的如何设置ACLs信息的,请查看ACL文档.

#### ACLs

这个监测是有且仅有的一次对ACL的操作,如果Consul服务器的配置中没有规定`acl.tokens.master`,


This endpoint does a special one-time bootstrap of the ACL system, making the first management token if the acl.tokens.master configuration entry is not specified in the Consul server configuration and if the cluster has not been bootstrapped previously. This is available in Consul 0.9.1 and later, and requires all Consul servers to be upgraded in order to operate.

This provides a mechanism to bootstrap ACLs without having any secrets present in Consul's configuration files.

[1]: https://www.consul.io/api/acl/acl.html#bootstrap-acls
[2]: https://www.consul.io/api/acl/acl.html#check-acl-replication
[3]: https://www.consul.io/api/acl/acl.html#translate-rules