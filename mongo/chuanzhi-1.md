# 【传智播客】MongoDB分片


## 概念：分片&复制集

分片支持水平扩展，是指将数据拆分，将数据分散在不同的机器上；
复制集是纵向扩展，保证数据备份，同步保证读写分离；

![](http://www.runoob.com/wp-content/uploads/2013/12/sharding.png)


shard: 分片，数据节点，每个分片可以部署成复制集
mongos: 查询路由，提供连接在客户端和分片集群之间
config servers: 配置服务器，提供集群的元数据和配置设置，必须是复制集而且这个复制集不能有投票几点

## 实现原理

MongoDB分片的基本思想就是将集合切分成小块。这些块分散到若干片里面，每个片只负责总数据的一部分。
应用程序不必知道哪片对应哪些数据，甚至不需要知道数据已经被拆分了，所以在分片之前要运行一个路由进程，该进程名为`mongos`。
这个路由器知道所有数据的存放位置，所以应用可以连接它来正常发送请求。对应用来说，它仅知道连接了一个普通的mongod。
路由器知道数据和片的对应关系，能够转发请求到正确的片上。如果请求有了回应，路由器将其收集起来回送给应用。

## 分片

设置分片时，需要从集合里面选一个键，用该键的值作为数据拆分的依据。这个键称为片键(shard key)。
{name:"zhangsan",age:1}
用个例子来说明这个过程：假设有个文档集合表示的是人员。
如果选择名字("name")作为片键，第一片可能会存放名字以A~F开头的文档，
第二片存的G~P的名字，第三片存的Q~Z的名字。
随着添加或者删除片，MongoDB会重新平衡数据，使每片的流量都比较均衡，数据量也在合理范围内。

        client → mongos （连接进程）→ | -- mongod（数据节点）
                            ↓↑                                       | -- mongod（数据节点）
                           config （元数据节点）    | -- mongod（数据节点）
                           
                           
                           
-----

            client → mongos （连接进程）←→config （元数据节点）
                                    ↑↓                                  
                ------------------------------                   
                |                     |                     |                             
      mongod       mongod     mongod                         
                                         
-----




### 1. 创建数据目录，启动元数据节点配置节点

1. config-server必须是复制集
2. config-server不支持arbiter

配置文件
```
dbpath=shards/config-1-d
logpath=shards/config-1-log/mongod.log
fork=true
bind_ip_all=true
logappend=true
auth=false
port=8887
oplogSize=1024
journal=true
replSet=imooc
configsvr=true
```
启动

```
bin/mongod -f shards/config-3.conf
```

客户端链接配置服务器(任意一台),配置复制集(初始化复制集)

```
bin/mongo localhost:8887

config = { "_id":"imooc",members:[{"_id":0,"host":"127.0.0.1:8887"}, {"_id":1,"host":"127.0.0.1:8888"},{"_id":2,"host":"127.0.0.1:8889"}] }

rs.initiate(config)

#可以写成下面这一行
#rs.initiate( { "_id":"imooc",members:[{"_id":0,"host":"127.0.0.1:8887"}, {"_id":1,"host":"127.0.0.1:8888"},{"_id":2,"host":"127.0.0.1:8889"}] })

```
注意这里不能设置投票节点,配置服务器存储元数据,和数据服务器的复制集不一样,这里必须要求配置服务器是复制集,还不能为投票节点`"arbiterOnly" : true`不能设置


### 2. 启动mongos进程，连接元数据节点

mongos是路由进程,提供客户端链接,客户端连接走路由,
mongos启动通过configbd指明元数据服务器

配置文件
```
port=5555
logpath=mongos.log
fork=true
configdb=imooc/localhost:8887,localhost:8888,localhost:8889
```

启动

```
 bin/mongos -f mongos.conf
```

安装过程会遇到一些问题,杀死mongo进程用kill -2 







### 3. 创建数据目录，分别启动数据节点
配置文件
```
dbpath=shards/1111-d
logpath=shards/1111-log/mongod.log
fork=true
bind_ip_all=true
logappend=true
auth=false
port=1111
shardsvr=true
```
启动
`bin/mongod -f shards/3333.conf`


### 4. 启动客户端连接mongos

`mongo 127.0.0.1:5555/admin`

### 5. 分别将分片节点添加到元数据节点（配置中心）

  添加分片，如果分片也是复制集仅需添加主服务器即可
  
  严格按照以下4个步骤执行
  
步骤1：添加分片`db.runCommand({"addshard":"localhost:4444","allowLocal":true})`查看状态`sh.status()`
步骤2：指定数据要保存的数据库`db.runCommand({"enablesharding":"test"})`也可以使用` sh.enableSharding("test")`
步骤3：为分片字段建立索引,片键必须是索引`db.user.ensureIndex({"name":"hashed"})`
步骤4：指定分片集合和片键`db.runCommand({"shardcollection":"test.user","key":{"name":"hashed"}})`,也可以使用`sh.shardCollection(“test.user”,{"name":”hashed”})`





### 6. 测试

db.printShardingStatus()

```
mongos> use test
switched to db test
mongos> db
test
mongos> for(i=1;i<100;i++)db.user.insert("name":"woms"+i)
```
分别登录到三个分片服务器上查看存储的数量

1号机器
```
[root@master mongodb]# bin/mongo localhost:1111
MongoDB shell version v3.6.2
connecting to: mongodb://localhost:1111/test
MongoDB server version: 3.6.2
>  db.user.count()
24
> 
```

2号机器
```
connecting to: mongodb://localhost:2222/test
MongoDB server version: 3.6.2
> db.user.count()
35
> 

```

3号机器
```
[root@master mongodb]# bin/mongo localhost:3333/test
MongoDB shell version v3.6.2
connecting to: mongodb://localhost:3333/test
MongoDB server version: 3.6.2
> db.user.count()
40
> 
```                                 
                                         
                                         

