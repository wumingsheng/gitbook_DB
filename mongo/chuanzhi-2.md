# 【传智播客】MongoDB复制集


![](https://docs.mongodb.com/manual/_images/replica-set-read-write-operations-primary.bakedsvg.svg)

单台计算机新能不够，并发有限，我们可以利用多台计算机对外 提供服务的时候，我们能够在处理客户端的时候他的这样
的一个并发数量能够达到比较均衡的这样的需求

主节点写数据，从节点读数据，读写分离
从节点从主节点同步数据

复制集和主从最大的区别就是支持容灾和故障切换,自动选举出主节点,主节点只有一个
当存活的节点等于小于一半节点,不再选举主节点,所有节点降级为secondary节点,此时只支持读操作,不再支持写操作

 ## 搭建复制集
 
 - 1个primary(主节点)
 - 1个secondary(从节点)
 - 1个arbiter(投票节点)

### 1. 配置文件

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
```

### 2. 根据配置文件启动三个实例

`bin/mongod -f shards/config-3.conf`

```
 ps -ef|grep mongo
root       5356      1  1 08:19 ?        00:00:01 bin/mongod -f shards/config-1.conf
root       5388      1  0 08:20 ?        00:00:00 bin/mongod -f shards/config-2.conf
root       5416      1  0 08:20 ?        00:00:00 bin/mongod -f shards/config-3.conf
root       5444   5019  0 08:21 pts/0    00:00:00 grep --color=auto mongo
[root@node1 mongodb-linux-x86_64-rhel70-3.6.3]# 
```

### 3. 客户端登录

```
bin/mongo localhost:8887/admin
> db
admin
```

### 4. 编写复制集初始化的配置文件

定义config对象
```
> config = {
... "_id":"imooc",
... members:[{"_id":0,"host":"127.0.0.1:8887"}, {"_id":1,"host":"127.0.0.1:8888"},{"_id":2,"host":"127.0.0.1:8889"}]
... }
```

将其中的一个节点更改为投票节点,投票节点不存储数据,只起到投票选举的作用

```
> config.members[2]
{ "_id" : 2, "host" : "127.0.0.1:8889" }
> config.members[2] = {"_id":2,"host":"127.0.0.1:8889","arbiterOnly":true}
{ "_id" : 2, "host" : "127.0.0.1:8889", "arbiterOnly" : true }
> 
```

```
> config  #重新观看config对象
{
        "_id" : "imooc",
        "members" : [
                {
                        "_id" : 0,
                        "host" : "127.0.0.1:8887"
                },
                {
                        "_id" : 1,
                        "host" : "127.0.0.1:8888"
                },
                {
                        "_id" : 2,
                        "host" : "127.0.0.1:8889",
                        "arbiterOnly" : true
                }
        ]
}
> rs.initiate(config)  # 通过对象初始化复制集
{
        "ok" : 1,   # 1,证明成功
        "operationTime" : Timestamp(1519652253, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1519652253, 1),
                "signature" : {
                        "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
                        "keyId" : NumberLong(0)
                }
        }
}
imooc:SECONDARY> # 这里也表明成功
imooc:PRIMARY> rs.status()   #查看复制机的状态,rs.help()可以查看rs的所有函数
```

### 5. 登录其他两个服务器,查看状态进行验证

```
#从节点

bin/mongo localhost:8888/test
MongoDB shell version v3.6.3
connecting to: mongodb://localhost:8888/test
MongoDB server version: 3.6.3
imooc:SECONDARY> 

# 投票节点

[root@node1 mongodb-linux-x86_64-rhel70-3.6.3]# bin/mongo localhost:8889/test
MongoDB shell version v3.6.3
connecting to: mongodb://localhost:8889/test
MongoDB server version: 3.6.3
imooc:ARBITER> 
```

### 6. 在主节点插入数据,看从节点的是否有数据

主节点插入数据

```
mooc:PRIMARY> 
imooc:PRIMARY> db
admin
imooc:PRIMARY> use test
switched to db test
imooc:PRIMARY> db
test
imooc:PRIMARY> db.imooc.insert({"name":"imooc"})
WriteResult({ "nInserted" : 1 })
imooc:PRIMARY> 
```
从节点查看数据,注意`rs.slaveOK(1)`
```
imooc:SECONDARY> rs.slaveOk(true)
imooc:SECONDARY> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
imooc:SECONDARY> db
test
imooc:SECONDARY> show collections
imooc
imooc:SECONDARY> db.imooc.find()
{ "_id" : ObjectId("5a94106ae4c4ddec5e2d389a"), "name" : "imooc" }
imooc:SECONDARY> 
```
查看投票节点,没有数据

``
imooc:ARBITER> rs.slaveOk(1)
imooc:ARBITER> show dbs
local  0.000GB
imooc:ARBITER> `
``



## 功能区分

主节点: 提供读写服务的节点
从节点: 提供读服务的节点
	| -- 隐藏节点:对程序不可见的节点
	| -- 延时节点:延时复制节点
	| -- 投票节点:具有投票权的节点,不上arbiter
投票节点:atbiter节点,无数据,仅作选举和充当复制集节点,也称为选举节点
















