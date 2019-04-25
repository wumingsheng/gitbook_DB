# 【运维】set模式

## 1. 搭建运行三个单节点

可以参考单节点搭建步骤：

配置文件使用如下：

```properties
dbpath=data
logpath=log/mongod.log
fork=true
bind_ip_all=true
logappend=true
auth=false
port=27017
oplogSize=1024
journal=true
replSet=univer
```

官方配置文件选项解释如下：

https://docs.mongodb.com/manual/reference/configuration-options/#configuration-file

2.6之后引入yaml格式，老格式依然向后兼容，使用yum安装生成的默认配置文件

```yaml
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# Where and how to store data.
storage:
  dbPath: /var/lib/mongo
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1  # Enter 0.0.0.0,:: to bind to all IPv4 and IPv6 addresses or, alternatively, use the net.bindIpAll setting.


#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options

#auditLog:

#snmp:
```



## 2. 三个单节点的配置成replica set

### 2.1 定义配置集群信息

```bash
{
    "_id": "univer",
    "members": [{
        "_id": 0,
        "host": "10.23.102.123:27017"
    }, {
        "_id": 1,
        "host": "10.23.102.68:27017"
    }, {
        "_id": 2,
        "host": "10.23.102.200:27017",
        "arbiterOnly": true
    }]
}
```



### 2.2 初始化rs集群

```bash
rs.initiate(config) 
rs.status()    # 查看状态
```



## 3. 验证

主节点插入数据，从节点查询数据

```bash
rs.slaveOk(true)  # 从节点默认是不允许读写的，查询数据之前执行这条命令
```





# 官方搭建步骤

1. Start each member of the replica set with the appropriate options
2. Connect a mongo shell to one of the mongod instances
3. Initiate the replica set
4. View the replica set configuration
5. Ensure that the replica set has a primary


## 1. 启动每一个节点用合适的配置选项

```yaml
# 日志相关配置
systemLog:
  destination: file
  path: /opt/mongodb-linux-x86_64-rhel70-4.0.4/log/mongod.log  # 日志输出路径
  logAppend: true

# 存储相关配置
storage:
  dbPath: /opt/mongodb-linux-x86_64-rhel70-4.0.4/data/   #指定数据存储的位置
  journal:
    enabled: true

# 进程相关配置
processManagement:
  fork: true  # fork and run in background 以daemon模式启动mongod
  pidFilePath: /opt/mongodb-linux-x86_64-rhel70-4.0.4/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo

# 网络相关配置
net:
  port: 27017 # 运行端口
  bindIp: 0.0.0.0  # 绑定ip，用逗号隔开，允许所有地址访问：0.0.0.0，另外可以使用net.bindIpAll允许所有地址访问

# 安全相关配置
security:
  authorization: enabled # enabled
  keyFile: /opt/mongodb-linux-x86_64-rhel70-4.0.4/autokey

#operationProfiling:

#副本集相关配置
replication:
  replSetName: rs0

#sharding:

## Enterprise-Only Options

#auditLog:

#snmp:
```



和单节点配置项相比，主要是添加了`replication.replSetName`配置项



```bash
openssl rand -base64 756 > /opt/mongodb-linux-x86_64-rhel70-4.0.4/autokey  #生成一个加密串，其实只要内容相同就可以，这里使用工具生成了一个字符串
chmod 400 /opt/mongodb-linux-x86_64-rhel70-4.0.4/autokey # 设置只读权限，免得被无意修改了
scp autokey root@10.23.102.68:$(pwd) # 复制给其他两个节点
```





```bash
bin/mongod -f conf/mongod.conf
```



## 2. 连接一个mongo-shell到其中一个实例

```bash
 bin/mongo
```



## 3. 初始化副本集

```
rs.initiate( {
   _id : "rs0",
   members: [
      { _id: 0, host: "10.23.102.123:27017" },
      { _id: 1, host: "10.23.102.68:27017" },
      { _id: 2, host: "10.23.102.200:27017" }
   ]
})
```



## 4. 查看副本集配置

```yaml
rs0:PRIMARY> db    # 查看当前db
test
rs0:PRIMARY> use admin  #切换db
switched to db admin
rs0:PRIMARY> db
admin
rs0:PRIMARY> db.createUser({ user: "root", pwd: "123456", roles: [{ role: "root", db: "admin" }] })   # 创建用户，给root角色
Successfully added user: {
        "user" : "root",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}
rs0:PRIMARY> db.auth("root","123456")  # 查看用户角色是否成功
1
rs0:PRIMARY> rs.conf()
```



## 5. 查看集群状态

```bash
rs.status()
```



## 6. 验证数据

```bash
# master插入数据
rs0:PRIMARY> use test
switched to db test
rs0:PRIMARY> db.user.insert({"name":"123456"})
WriteResult({ "nInserted" : 1 })
rs0:PRIMARY> db.user.find()
{ "_id" : ObjectId("5c17be0959c694bdb2d421ca"), "name" : "123456" }
rs0:PRIMARY> show tables
user
# secondary查询数据
rs0:SECONDARY> use admin # 先登录到admin认证
switched to db admin
rs0:SECONDARY> db.auth("root","123456")
1
rs0:SECONDARY> rs.slaveOk(true) # 打开slave查询功能
rs0:SECONDARY> use test # 切换到test数据库查询数据
switched to db test
rs0:SECONDARY> db.user.find()
{ "_id" : ObjectId("5c17be0959c694bdb2d421ca"), "name" : "123456" }
```






