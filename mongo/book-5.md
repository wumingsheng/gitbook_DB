# 【第五章】MongoDB的安全

# 第五章：mongoDB的安全

标签（空格分隔）： mongoDB

---

1. mongoDB安全概览
2. 物理隔离与网络隔离
3. ip白名单隔离
4. 用户名密码鉴权

## mongoDB安全概览

1. 最安全是物理隔离：不现实
2. 网络隔离其次
3. 防火墙再其次
4. 用户名和密码在最后

默认mongoDB不开启权限安全认证，开启权限安全认证有下面两种方式：

1. auth开启
2. keyfile开启

### auth开启方式

1. 修改启动配置文件，添加参数开启权限认证
```
port = 12345
#可以使用相对路径也可以使用绝对路径，这里使用相对路径
dbpath = data
logpath = log/mongod.log
fork = true
#开启权限认证
auth = true
```
2. 添加用户`createUser`
```json
{user:"<username>",pwd:"<password>",customData:{<info>},roles:[{role:"<role>",db:"<database>"}]}
```
角色类型：内建类型（read < readWrite < dbAdmin < dbOwner < userAdmin）

```shell
> use admin
switched to db admin
> db.createUser({user:"root",pwd:"123456",customData:{age:18,addr:"shanxi"},roles:[{role:"userAdmin",db:"admin"}]})
Successfully added user: {
	"user" : "root",
	"customData" : {
		"age" : 18,
		"addr" : "shanxi"
	},
	"roles" : [
		{
			"role" : "userAdmin",
			"db" : "admin"
		}
	]
}
> 

```

### mongoDB用户角色详解
1. 数据库角色（read,readWrite,dbAdmin,dbOwner,userAdmin）
2. 集群角色（clusterAdmin,clusterManager...）
3. 备份角色（backup,restore...）
4. 其他特设权限（DBAdminAnyDatabase...）