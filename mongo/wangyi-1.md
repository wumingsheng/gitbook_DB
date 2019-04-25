# 【网易云课堂】MongoDB简介


MongoDB数据库是一种NoSQL数据库

传统方式：数据表 → JDBC读取 → POJO(VO,DO) → 控制层转换为JSONS数据 → 客户端
现在方式：直接有一个数据库存在要显示的json数据，省略了所有需要转换的过程

其中NoSQL数据数据库负责数据的读取，因为直接保存的就是json（前提：MongoDB中的数据是排列好的组合数据）

例如：现在要求显示出每个雇员的编号，姓名，职位，部门名称，部门职位，工资等级。
传统的数据库一定存放大量的冗余数据，不合理，有了NoSQL数据库，可以直接在业务层里面将数据按照指定的结构进行存储。

在MongoDB数据库中，与传统数据库有如下的概念对应：

|关系型数据库     |    MongoDB                            |
| -----         |  -----                                |
|数据库          |      数据库（类似与mysql，非oracle）     |
|表             |       集合                             |
|行             |       文档                             |
|列             |       成员                             |
|主键           |      object ID（自动维护）               |

MongoDB面向集合的存储过程，模式自由（无模式），方便的进行存储扩充，支持索引，支持短暂数据保留，具备完整的数据库状态的监控，基于BSON（mongo自己的json）应用。

## mongoDB的安装与配置

www.mongodb.org直接下载可用版本.tar.gz并解压

```bash
tar -zxvf .tar.gz -C /opt
```

如果想要正常启动mongoDB，必须先创建一个存放数据文件的目录，默认是/data/db

```bash
--dbpath arg                          directory for datafiles - defaults to /data/db
```

bin目录下每个二进制文件的作用可以看README

服务端启动是mongod文件，./mongod可以看启动命令

mongoDB数据库服务端进程启动可以通过指定配置文件的方式指定启动参数，也可以通过后面直接配置启动参数两种方式启动，进入bin目录下，运行`$ ./mongod --help`，这里详细介绍了启动参数含义和默认值

```bash
mkdir log db conf
cd conf
touch mongod.conf
bin/mongod -f conf/mongod.conf
```


配置文件
```properties
#可以使用相对路径也可以使用绝对路径，这里使用相对路径
dbpath=db
#启动日志
logpath=log/mongod.log
#后台启动
fork=true
#security
auth=false
#允许所有的客户端连接
bind_ip_all=true
```

关闭后台服务端

- `pkill mongod`，视乎必须要在bin目录下执行才有效
- mongo客户端关闭
```bash
use admin 
db.shutdownServer()
```

### MongoDB的基础操作

MongoDB没有表结构，没有字段类型，没有事物，保存的数据结构就是json结构，只不过在操作数据的时候才用到MongoDB自己的一些操作符


使用、创建数据库`use mldn`(实际上这个时候并不会创建数据库，只有创建 了表之后才会自动创建数据库)
显示所有的数据库` show databases`
创建表` db.createCollection("emp")`
查看所有的表集合`show collections`
删除集合（表）语法：db.表明.drop()
删除当前数据库 语法：db.dropDatabase()
查看当前所在的数据库：db
查看针对数据库的操作：db.help()

但是很多时候mongodb使用都是直接向里面保存数据，不用先创建表，在保存数据的时候会自动创建表（无模式）
```bash
# 插入数据
> db.emp.insert({"deptno":10,"dname":"caiwubu","loc":"beijing"})
WriteResult({ "nInserted" : 1 })
# 查询数据db.表明.find({查询条件})
> db.emp.find()
{ "_id" : ObjectId("5a8f7d61e9efc2d31c1c3577"), "deptno" : 10, "dname" : "caiwubu", "loc" : "beijing" }

```

插入数据的另一种写法
```
var deptData = {
  
  "d_no":"3652",
  "d_name":"研发部",
  "d_loc":"深圳",
  "count":20,
  "avg":8000.0

}
db.dept.insert(deptData)
```


关于ID的问题

在MongoDB集合（表）中，每一行记录都会自动的生成一个id，`_id" : ObjectId("5a8f7e2de9efc2d31c1c3578")`
数据组成是：时间戳 + 机器码 + pid + 计数器 ，这个ID的信息是MongoDB自己为我们生成的

查看一条记录，MongoDB是严格要求大小写的，大小写敏感
```
> db.dept.findOne()
{
        "_id" : ObjectId("5a8f7e2de9efc2d31c1c3578"),
        "deptno" : 10,
        "dname" : "caiwubu",
        "loc" : "beijing"
}
```

删除数据，条件按照json写出

```
db.dept.remove({ "_id" : ObjectId("5a8f7e2de9efc2d31c1c3578")})
```

跟新数据
```
var deptData = {
  
  "d_no":"3652",
  "d_name":"研发2部",
  "d_loc":"深圳",
  "count":50,
  "avg":8000.0

}

db.dept.update({"_id" : ObjectId("5a8f7face9efc2d31c1c3579")},deptData)
```











