# 【运维】stand-alone模式

https://www.jianshu.com/p/79caa1cc49a5

## 安装

   ```bash
   wget mongo.tar
   tar -zxvf mongo.tar
   cd mongo
   mkdir log data conf
   touch conf/mongod.conf
   ```

## 配置

   ```bash
   vi conf/mongod.conf
   ```



   ```conf
   port=27017
   dbpath=data
   logpath=log/mongod.log
   fork=true   # 后台启动
   bind_ip_all=true   #允许所有的ip访问
   auth=true   # 开启用户名密码认证
   ```

## 启动

   ```bash
   bin/mongod -f conf/mongod.conf
   ```

## 关闭

   ```bash
   pkill mongod
   
   ```

## 开启安全认证

开启安全认证必须同时满足以下两个条件：

1. mongod启动以auth参数启动
2. 数据库中创建了user

###  1. 对所有的dbs全局认证

```bash
# 创建超级管理员,超级管理员的信息只保存在admin数据库中，超级管理员登录可以对所有的数据库进行操作
use admin
db.createUser({ user: "root", pwd: "123456", roles: [{ role: "root", db: "admin" }] })
db.auth("root","123456")   # 验证是否成功 返回1说明成功
```

现在有两种方式进行用户身份的验证


```bash
# 方式一
bin/mongo --port 27017 -u "root" -p "123456" --authenticationDatabase "admin"


# 方式二
bin/mongo --port 27017
use admin
db.auth("root", "123456")    #  输出 1 表示验证成功
```



### 2. 对具体的数据库db认证

```bash
use test #在哪个数据库中创建的用户，就要用哪个数据库验证，用户的信息跟随数据库
db.createUser(
  {
    user: "wms",
    pwd: "123456",# 这里最好给四个基本role
    roles: [ { role: "read", db: "test" },{ role: "readWrite", db: "test" },
    		 { role: "dbAdmin", db: "test" },{ role: "userAdmin", db: "test" } ]
  }
)
bin/mongo -u wms -p 123456 --authenticationDatabase test 
show tables



# 更新用户
db.updateUser("wms",
  {pwd: "123456",
    roles: [ { role: "read", db: "test" },{ role: "readWrite", db: "test" },
    { role: "dbAdmin", db: "test" },{ role: "userAdmin", db: "test" } ]
  }
)
```

> 还有一点需要注意，如果 admin 库没有任何用户的话，即使在其他数据库中创建了用户，启用身份验证，默认的连接方式依然会有超级权限

## 内建role

- read：允许用户读取指定数据库
- readWrite：允许用户读写指定数据库
- dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
- userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
- clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
- readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
- readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
- userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
- dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
- root：只在admin数据库中可用。超级账号，超级权限



> 1. 只在admin数据库中可用，意思是数据保存在admin数据库中，在其它数据库中不可用保存数据
> 2. 在那个数据库中保存的信息（可用），只能在那个数据库下验证，在其它的数据库下验证不了。也就是说有些role，是在admin下操作的，需要在admin下验证，但是可用对说有的db进行操作

## 使用yum安装

ali镜像源

```properties
[mongodb-org]
name=MongoDB Repository
baseurl=http://mirrors.aliyun.com/mongodb/yum/redhat/7Server/mongodb-org/3.2/x86_64/
gpgcheck=0
enabled=1
```

```bash
yum install -y mongodb-org
service mongod start
chkconfig mongod on
```

配置文件在/etc/mongod.conf




