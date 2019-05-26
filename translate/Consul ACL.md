
## ACLs

### ACL HTTP API

`/acl endPoint` 是用来管理Consul的ACL令牌和策略的,[使用ACL][1],[检查ACL复制状态][2]并且[交换规则][3]。有独立的页面针对`/acl endpoints`进行令牌和策略的管理。

对于更多的如何设置ACLs信息的,请查看ACL文档。

#### Bootstrap ACLs

这个监测是有且仅有的一次对ACL的操作，如果Consul服务器的配置中没有规定`acl.tokens.master`的配置或者集群没有初始化过。`Consul 0.9.1`版本和以上版本,需要所有的服务器都要升级到版本便于操作。

提供了一种不需要在配置文件中设置任何存在密码相关信息的机制。

Method| Path| Produces
:-:|:-:|:-:
PUT| /acl/bootstrap| application/json

下面的表格显示了这个检测端点对阻塞查询、一致性模式保持、代理缓存以及是否需要ACL

Blocking Queries|	Consistency Modes|	Agent Caching|	ACL Required
:-:|:-:|:-:|:-:
NO|	none|	none|	none

##### Sample Request
```
$ curl \
    --request PUT \
    http://127.0.0.1:8500/v1/acl/bootstrap
```

>废弃属性 - id属性是为了处理遗留下来的兼容性而存在的，主要是SecretID字段赋值。在以后的Consul版本的应用中，应该忽略ID这个字段属性，而不是作为移除服务的唯一字段。

##### Sample Response
```json5
{
    "ID": "527347d3-9653-07dc-adc0-598b8f2b0f4d",
    "AccessorID": "b5b1a918-50bc-fc46-dec2-d481359da4e3",
    "SecretID": "527347d3-9653-07dc-adc0-598b8f2b0f4d",
    "Description": "Bootstrap Token (Global Management)",
    "Policies": [
        {
            "ID": "00000000-0000-0000-0000-000000000001",
            "Name": "global-management"
        }
    ],
    "Local": false,
    "CreateTime": "2018-10-24T10:34:20.843397-04:00",
    "Hash": "oyrov6+GFLjo/KZAfqgxF/X4J/3LX0435DOBy9V22I0=",
    "CreateIndex": 12,
    "ModifyIndex": 12
}
```

你可以通过检测响应code判断是否有操作干扰ACL的正常引导。`200`表示正常启动成功，`403`表示集群已经启动过了，这种必须考虑到的集群的折中状态。

返回的token是可以无限次的对系统的细节进行管理。也可以用来配置以后的ACL系统。更多细节查看ACL指南。

### Check ACL Replication
这个监控返回值是ACL复制进程在数据中心的状态。用来运维者或者自动化进行检查ACL复制的运行状态。

有关更多详细信息，请参阅ACL复制指南。

Method| Path| Produces
:-:|:-:|:-:
GET| /acl/replication| application/json

Blocking Queries|	Consistency Modes|	Agent Caching|	ACL Required
:-:|:-:|:-:|:-:
NO|	consistent|	none|	none

#### Parameters
dc (string: "") - 指定查询的数据中心名称，如果不指定会查询默认的代理，指定方式是在URL拼接上参数。

#### Sample Request
```
$ curl \
    --request GET \
    http://127.0.0.1:8500/v1/acl/replication
```

#### Sample Response
```json
{
  "Enabled": true,
  "Running": true,
  "SourceDatacenter": "dc1",
  "ReplicationType" : "tokens",
  "ReplicatedIndex": 1976,
  "ReplicatedTokenIndex": 2018,
  "LastSuccess": "2018-11-03T06:28:58Z",
  "LastError": "2016-11-03T06:28:28Z"
}
```
+ Enabled - 报告数据中心是否可用
+ Running - 报告ACL复制是否正在运行，这个进程大约发生在leader选举后的60s后。
+ SourceDatacenter - 这里的认证ACL数据中心是从主数据中心的配置中复制出来的ACL列表。
+ ReplicationType - 当前使用的复制的方式
    + legacy - ACL复制传统模式并且正在复制传统的ACL tokens
    + policies - 在token复制被禁用的情况下的单纯复制ACL策略
    + tokens - ACL复制令牌和策略
+ ReplicatedIndex - 上一个复制成功的索引，具体复制的是什么数据需要根据复式方式来判断。对于传统模式会对`/v1/acl/list`监测返回的数据中`X-Consul-Index`头信息进行比较，用来判断是否获取到所有可用的ACL。对于令牌或者策略的方式下可以对`/v1/acl/polcies`监测的数据的`X-Consul-Index`的标头进行比较，判断是否获取到所有可用的ACL。
+ ReplicatedTokenIndex - 最后一个成功复制的token的索引。可以对`/v1/acl/tokens`监测的数据的`X-Consul-Index`的标头进行比较，判断是否获取到所有可用的tokens。请注意，ACL复制速率受限，因此索引可能会滞后于主数据中心。
+ LastSuccess - 上次同步操作成功的UTC时间。因为ACL复制是阻塞方式查询的，如果没有5min中内没有ACL复制的修改不会进行更新，如果没有同步成功显示`0001-01-01T00:00:00Z`
+ LastError - 上次同步操作失败的UTC时间。假设这个时间比上次成功时间要晚，说明复制过程出现了问题，如果没有出现错误则是`0001-01-01T00:00:00Z`。

### Translate Rules

>废弃属性 - 这个端点是用来出来1.4.0版本前的迁移问题的ACL系统的。以后的版本会移除对于传统ACL支持。

这个端点是用来将旧的语法转换为新的语法，用来管理Consul ACL的操作以及新旧令牌的执行策略。

Method| Path| Produces
:-:|:-:|:-:
POST| /acl/rules/translate| text/plain

Blocking Queries|	Consistency Modes|	Agent Caching|	ACL Required
:-:|:-:|:-:|:-:
NO|	consistent|	none|	acl:read

##### Sample Payload
```
agent "" {
   policy = "read"
}
```

##### Sample Request
```
$ curl -X POST -d @rules.hcl http://127.0.0.1:8500/v1/acl/rules/translate 
```

##### Sample Response
```
agent_prefix "" {
   policy = "read"
}
```

### Translate a Legacy Token's Rules

>废弃属性 - 这个端点是用来出来1.4.0版本前的迁移问题的ACL系统的。以后的版本会移除对于传统ACL支持。

这个端点是用来将嵌入式的传统的ACL添加到最新的语法中。
This endpoint translates the legacy rules embedded within a legacy ACL into the latest syntax. It is intended to be used by operators managing Consul's ACLs and performing legacy token to new policy migrations. Note that this API requires the auto-generated Accessor ID of the legacy token. This ID can be retrieved using the /v1/acl/token/self endpoint.

### Login to Auth Method
### Logout from Auth Method


[1]: https://www.consul.io/api/acl/acl.html#bootstrap-acls
[2]: https://www.consul.io/api/acl/acl.html#check-acl-replication
[3]: https://www.consul.io/api/acl/acl.html#translate-rules