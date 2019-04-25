# 【运维】shard-cluster模式

查询路由、配置服务器、分片
其中查询路由为mongos进程
配置服务器(默认端口27019)和分片(27018)都是mongod进程
配置服务器和分片都采取副本集（replica set）来确保可用性和健壮性，每个副本集最少包含三个节点。
查询路由都是单实例运行，没有副本集机制，可根据需要增加实例数量

所有mongod进程和mongos进程要使用同一个keyfile，保证认证可用


https://docs.mongodb.com/manual/tutorial/deploy-sharded-cluster-hashed-sharding/index.html

![](assets/深度截图_选择区域_20181218180412.png)





![](assets/深度截图_选择区域_20181218180619.png)





## 1. 配置config server副本集

```bash
tar -zxvf mongodb-linux-x86_64-rhel70-4.0.4.tgz -C /opt/
cd /opt/
mv mongodb-linux-x86_64-rhel70-4.0.4/ mongodb
cd mongodb/
rm -rf LICENSE-Community.txt MPL-2 README THIRD-PARTY-NOTICES
mkdir data conf log
touch conf/mongod.conf
openssl rand -base64 756 > autokey
chmod 400 autokey
scp -r mongodb root@10.23.102.76:/opt/
scp -r mongodb root@10.23.102.119:/opt/
```

```yaml
sharding:
  clusterRole: configsvr
replication:
  replSetName: confsrv
systemLog:
  destination: file
  path: /opt/mongodb/log/mongod.log
  logAppend: true
net:
  port: 27019
  bindIp: 0.0.0.0
storage:
  dbPath: /opt/mongodb/data/
  directoryPerDB: true
processManagement:
  fork: true
  pidFilePath: /opt/mongodb/mongod.pid
security:
  authorization: enabled
  keyFile: /opt/mongodb/autokey
```

```bash
cd /opt/mongodb/
bin/mongod -f conf/mongod.conf
```

连接并初始化


```bash
bin/mongo --port 27019

rs.initiate( 
    { 
        _id:"confsrv",
        configsvr: true,
        members:[{"_id":0,"host":"10.23.102.110:27019"}, {"_id":1,"host":"10.23.102.119:27019"},{"_id":2,"host":"10.23.102.76:27019"}] 
    }
)
rs.status()

```

## 2. 配置shard1副本集


```bash
$ tar -zxvf mongodb-linux-x86_64-rhel70-4.0.4.tgz -C /opt/
$ cd /opt
$ mv mongodb-linux-x86_64-rhel70-4.0.4/ mongodb
$ cd mongodb
$ rm -rf LICENSE-Community.txt MPL-2 README THIRD-PARTY-NOTICES 
$ mkdir data log conf 
$ touch conf/mongod.conf
$ scp -r mongodb root@10.23.102.165:/opt/
$ scp -r mongodb root@10.23.102.86:/opt/
$ bin/mongod -f conf/mongod.conf
$ ssh root@10.23.102.165 "cd /opt/mongodb && bin/mongod -f conf/mongod.conf"
$ ssh root@10.23.102.86 "cd /opt/mongodb && bin/mongod -f conf/mongod.conf"
$ bin/mongo --port 27018
rs.initiate( { "_id":"shard1",members:[{"_id":0,"host":"10.23.102.163:27018"}, {"_id":1,"host":"10.23.102.165:27018"},{"_id":2,"host":"10.23.102.86:27018"}] })
rs.status()
```

```yaml
sharding:
  clusterRole: shardsvr
replication:
  replSetName: shard1
  oplogSizeMB: 10240
systemLog:
  destination: file
  path: /opt/mongodb/log/mongod.log
  logAppend: true
net:
  port: 27018
  bindIp: 0.0.0.0
storage:
  dbPath: /opt/mongodb/data/
  directoryPerDB: true
processManagement:
  fork: true
  pidFilePath: /opt/mongodb/mongod.pid
security:
  authorization: enabled
  keyFile: /opt/mongodb/autokey
```

同上配置shard2副本集,注意修改副本集名称和ip



## 3. 配置mongos router实例

```yaml
sharding:
  configDB: confsrv/10.23.102.110:27019,10.23.102.119:27019,10.23.102.76:27019
systemLog:
  destination: file
  path: /opt/mongodb/log/mongos.log
  logAppend: true
processManagement:
  fork: true
  pidFilePath: /opt/mongodb/mongos.pid
net:
  port: 27017
  bindIp: 0.0.0.0
security:
  authorization: enabled
  keyFile: /opt/mongodb/autokey
```


```bash
cd /opt/mongodb/
bin/mongos -f conf/mongos.conf

```

连接到mongos

```bash
mongo --host <hostname> --port <port>
```

然后将上面创建的分片依次添加进来，添加分片，如果分片也是复制集仅需添加主服务器即可,执行：

```
sh.addShard( "shard1/10.23.102.163:27018")
sh.addShard( "shard2/10.23.102.136:27018")
```



操作完成后，可以查看分片集群状态，执行：

```
sh.status()
```


会发现每个分片除了添加的主节点外，从节点也自动加入了，后续分片副本集如果发生变化（增删节点）也会自动识别出来。



让数据库使用分片
```
sh.enableSharding("test")
```
为分片字段建立索引,片键(分片字段)必须是索引
```
db.user.createIndex({"name":"hashed"})
```
指定分片集合和片键
```
sh.shardCollection("test.user", { name : "hashed" } )
```



## 4. 测试

```
mongos> db.user.insert({"name":"a"})
WriteResult({ "nInserted" : 1 })
mongos> db.user.insert({"name":"b"})
WriteResult({ "nInserted" : 1 })
mongos> db.user.insert({"name":"c"})
WriteResult({ "nInserted" : 1 })
mongos> db.user.insert({"name":"d"})
WriteResult({ "nInserted" : 1 })
mongos> db.user.insert({"name":"e"})
WriteResult({ "nInserted" : 1 })
mongos> db.user.insert({"name":"f"})
WriteResult({ "nInserted" : 1 })
mongos> db.user.insert({"name":"g"})
WriteResult({ "nInserted" : 1 })
```

插入了n条数据, 用图形化客户端连接，所有的数据都可以看到，和单实例似乎没有区别
登录到每个片区，查看数据分片情况


```bash
$ bin/mongo --port 27018
shard2:PRIMARY> db
test
shard2:PRIMARY> db.user.find() # 片区2只存了4条数据
{ "_id" : ObjectId("5c18bbb06c260bdd399f1363"), "name" : "a" }
{ "_id" : ObjectId("5c18bbb36c260bdd399f1364"), "name" : "b" }
{ "_id" : ObjectId("5c18bbb76c260bdd399f1365"), "name" : "c" }
{ "_id" : ObjectId("5c18bbba6c260bdd399f1366"), "name" : "d" }
shard1:PRIMARY> db
test
shard1:PRIMARY> db.user.find() # 片区1存了3条数据
{ "_id" : ObjectId("5c18bbbe6c260bdd399f1367"), "name" : "e" }
{ "_id" : ObjectId("5c18bbc26c260bdd399f1368"), "name" : "f" }
{ "_id" : ObjectId("5c18bbc46c260bdd399f1369"), "name" : "g" }
```

分片集群对于客户端应用来说是透明的，客户端应用只需将分片集群视为单个mongod实例，所有客户端请求都连接分片集群中的路由（mongos）。

应用只需将原有Mongo配置指向路由即可，如果想要使用多个路由，可以将多个路由地址用逗号连接，类似副本集的配置

![深度截图_选择区域_20181218180412](assets/深度截图_选择区域_20181218180412.png)









## 5. 问题

-----



1. springboot怎么使用分片集群？自动使用分片数据库，创建索引，使用片键？

   网上没有找到相关资料，似乎要手动去维护了，要想使用分片集群，要预先设置好



-----









## 1. 环境准备



提供配置文件`conf/mongo.conf`公共配置项

```yaml
systemLog:
  destination: file
  path: /opt/mongodb/log/mongos.log
  logAppend: true
processManagement:
  fork: true
  pidFilePath: /opt/mongodb/mongos.pid
net:
  bindIp: 0.0.0.0
security:
  authorization: enabled
  keyFile: /opt/mongodb/autokey
```

在其中任意一个节点上创建基础环境，并拷贝给其他节点

```bash
tar -zxvf mongodb-linux-x86_64-rhel70-4.0.4.tgz -C /opt/
cd /opt/
mv mongodb-linux-x86_64-rhel70-4.0.4/ mongodb
cd mongodb/
rm -rf LICENSE-Community.txt MPL-2 README THIRD-PARTY-NOTICES
mkdir data conf log
touch conf/mongo.conf
vi conf/mongo.conf
openssl rand -base64 756 > autokey
chmod 400 autokey
cd ..
scp -r mongodb root@10.23.102.110:/opt/
scp -r mongodb root@10.23.102.119:/opt/
scp -r mongodb root@10.23.102.76:/opt/
scp -r mongodb root@10.23.102.163:/opt/
scp -r mongodb root@10.23.102.165:/opt/
scp -r mongodb root@10.23.102.86:/opt/
scp -r mongodb root@10.23.102.136:/opt/
scp -r mongodb root@10.23.102.45:/opt/
scp -r mongodb root@10.23.102.55:/opt/
scp -r mongodb root@10.23.102.110:/opt/
```



## 2. Create the Config Server Replica Set

添加额外配置项,如果有必要配置net.port,默认配置中心端口是27019

```yaml
sharding:
  clusterRole: configsvr
replication:
  replSetName: confsrv
storage:
  dbPath: /opt/mongodb/data/
  directoryPerDB: true
```

分别在config server的三个节点上执行，默认configsvr端口27019

```bash
cd /opt/mongodb/
vi conf/mongo.conf
bin/mongod -f conf/mongo.conf
```

连接config server任意一节点并初始化副本集，注意下面用到的_id是上面配置的副本集的name，ip是节点端口号

```bash
bin/mongo --port 27019

rs.initiate( 
    { 
        _id:"confsrv",
        configsvr: true,
        members:[{"_id":0,"host":"10.23.102.110:27019"},{"_id":1,"host":"10.23.102.119:27019"},{"_id":2,"host":"10.23.102.76:27019"}] 
    }
)
rs.status()
```

config-server不用配置管理员

## 3. Create the Shard Replica Sets



配置文件需要的额外配置项,shard默认端口27018，如果有必要配置net.port

```yaml
sharding:
  clusterRole: shardsvr
replication:
  replSetName: shard1
  oplogSizeMB: 10240
storage:
  dbPath: /opt/mongodb/data/
  directoryPerDB: true
```



在shard-1区上的三个节点上执行

```bash
 cd /opt/mongodb/
 vi conf/mongo.conf
 bin/mongod -f conf/mongo.conf 
```

初始化复制集

```bash
$ bin/mongo --port 27018
rs.initiate( { "_id":"shard1",members:[{"_id":0,"host":"10.23.102.163:27018"}, {"_id":1,"host":"10.23.102.165:27018"},{"_id":2,"host":"10.23.102.86:27018"}] })
rs.status()
```



同理，重复上述3步骤，扩展shard数量。注意修改复制集名称和ip



## 4. users

通常，要为分片集群创建用户，请连接到 [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)并添加分片群集用户。

但是，某些维护操作需要直接连接到分片群集中的特定分片(the specific shard)。要执行这些操作，必须直接连接到分片并作为分片本地(Shard-local)管理用户进行身份验证。

Shard-local用户仅存在于the specific shard中，并且只应用于the specific shard的维护和配置。您无法连接到[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)分享本地用户。



为指定分片创建Shard-local管理用户，只在分片副本集的primary节点上创建用户。

创建用户root/123456给超级管理员的权限

```bash
cd /opt/mongodb
bin/mongo --port 27018
rs.status()     # 找到primary节点，在primary节点上执行
admin = db.getSiblingDB("admin") #不用真实切换admin数据库（改变数据库的可用性），可以在amdin数据库上执行命令
admin.createUser(
  {
    user: "root",
    pwd: "123456",
    roles: [ { role: "root", db: "admin" } ]
  }
)
admin.auth("root","123456")   # 认证数据库
show dbs # 发现有权限了
```



可以在每个分片上都创建管理用户

```bash
shard2:PRIMARY> db #查询当前数据库是test
test
shard2:PRIMARY> show dbs  #查询所有的数据库，发现没有权限
shard2:PRIMARY> use admin #切换到admin数据库
switched to db admin
shard2:PRIMARY> db.createUser( #在admin上创建超级管理员
...   {
...     user: "root",
...     pwd: "123456",
...     roles: [ { role: "root", db: "admin" } ]
...   }
... )
Successfully added user: {
        "user" : "root",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}
shard2:PRIMARY> db.auth("root","123456")#用刚创建的超级管理员认证
1
shard2:PRIMARY> show dbs#认证完成以后，有权限了
admin   0.000GB
config  0.000GB
local   0.000GB
```





## 5. Connect a mongos to the Sharded Cluster



mongos节点的额外配置项，端口默认27017,如果有需要自己定义net.port

```yaml
sharding:
  configDB: <configReplSetName>/config-server-1:port,...
```



启动mongos进程，删掉security.authorization, mongos进程不支持这个配置

```bash
cd /opt/mongodb
bin/mongos -f conf/mongo.conf
Unrecognized option: security.authorization
try 'bin/mongos --help' for more information
# 把security.authorization删掉
sed -i '/authorization/d' conf/mongo.conf
bin/mongos -f conf/mongo.conf #再次执行
```



通过 localhost interface连接mongos任意一个实例

注意，这里因为启动了认证，所以必须运行mongo和mongos进程在相同的物理机上，localhost interface接口可以使用，因为到目前位置还没有为这个部署集群创建任何user,如果第一个user被创建后，localhost interface也将关闭。

```bash
bin/mongo --port 27017

```

创建user administrator，注意第一个用户必须具有`userAdminAnyDatabase`权限，我这里也给了一个root权限

```bash
mongos> db
test
mongos> use admin # 切换到admin数据库
switched to db admin
mongos> db
admin
mongos> db.createUser(  # 创建第一个集群超级管理员
  {
    user: "univer",
    pwd: "univer",
    roles: [ { role: "root", db: "admin" } ]
  }
)
mongos> db.auth("univer","univer") # 认证是否成功
1
mongos> show dbs
admin   0.000GB
config  0.000GB
```

根据需要可以添加不同角色用户（可选）

例如，为指定的数据库创建指定的username/password管理员账户

添加分片到集群

下面所有的操作，必须连接到mongos并使用全局集群管理员（mongos上创建的管理员）用户进行身份验证。

> This is the cluster administrator for the sharded cluster and *not* the shard-local cluster administrator.

指定副本集里面的任意一个节点就可以

```bash
# sh.addShard( "<replSetName>/s1-mongo1.example.net:27017")
sh.addShard( "shard1/10.23.102.169:27018")
sh.addShard( "shard2/10.23.102.252:27018")
```

如果添加的分片不是副本集只是一个单一节点，使用如下方式

```bash
sh.addShard( "s1-mongo1.example.net:27017")
```

重复执行知道添加所有的分片

=====================================================================

为数据库启用分片存储

```bash
# sh.enableSharding("<database>")
sh.enableSharding("test")
```



给表（Collection）分片

```bash
db.user.createIndex({"name":"hashed"})
sh.shardCollection("test.user", { name : "hashed" } )
# sh.shardCollection("<database>.<collection>", { <shard key> : "hashed" } )
```



hash分片和ranged分片

- hash分片写数据快，数据均衡分布
- ranged分片数据分布不均衡，查询很快，写数据慢

=====================================================================



添加mongos实例

使用已有的mongos实例的配置文件

启动进程,启动进程即可，不用作任何配置

```bash
cd /opt/mongodb
bin/mongos -f conf/mongo.conf
```





> 一定要区分shard-local管理员和mongos管理员的账户认证
>
> shard-local上创建的用户的指定片区的认证
>
> mongos上创建的用户是全局集群的认证


















