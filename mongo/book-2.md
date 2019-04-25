# 【第二章】MongoDB的安装与配置


## 安装启动

[mongoDB官网][1]

1. 在官网下载对应版本的二进制`.tar.gz`安装包并解压
2. 进入解压目录阅读`README`文件，这里介绍了每个文件的作用和启动方式
3. 进入bin目录下，运行`$ ./mongod --help`，这里详细介绍了启动参数含义和默认值
4. mongoDB数据库服务端进程启动可以通过指定配置文件的方式指定启动参数，也可以通过后面直接配置启动参数两种方式启动，这里演示一种通过配置文件的方式




```bash
# 创建mongoBD工作空间根目录
user@user-PC:~/Downloads$ mkdir mongodb_simple
user@user-PC:~/Downloads$ cd mongodb_simple/
# 在根目录下创建数据文件目录
user@user-PC:~/Downloads/mongodb_simple$ mkdir data
# 在根目录下创建日志文件目录
user@user-PC:~/Downloads/mongodb_simple$ mkdir log
# 在根目录下创建启动配置文件目录
user@user-PC:~/Downloads/mongodb_simple$ mkdir conf
# 在根目录下创建数据库启动二进制文件目录
user@user-PC:~/Downloads/mongodb_simple$ mkdir bin
# 拷贝解压安装包bin目录下服务端二进制启动文件到根工作空间的bin目录下
user@user-PC:~/Downloads/mongodb_simple$ cp ../mongodb-linux-x86_64-debian71-3.4.9/bin/mongod bin/
# 进入到配置文件目录
user@user-PC:~/Downloads/mongodb_simple$ cd conf
# 编写配置文件
user@user-PC:~/Downloads/mongodb_simple/conf$ vim mongod.conf
# 配置文件编写完毕后回到上级目录
user@user-PC:~/Downloads/mongodb_simple/conf$ cd ..
# 可以看到当前的目录结构如下
user@user-PC:~/Downloads/mongodb_simple$ ls
bin  conf  data  log
# 启动服务端 -f　指定配置文件
user@user-PC:~/Downloads/mongodb_simple$ ./bin/mongod -f conf/mongod.conf 
about to fork child process, waiting until server is ready for connections.
forked process: 7246
child process started successfully, parent exiting
# 查看日志信息，验证是否启动成功
user@user-PC:~/Downloads/mongodb_simple$ tail -f log/mongod.log
2017-10-18T09:38:41.921+0000 I NETWORK  [thread1] waiting for connections on port 12345
# 采用二进制安装包中的mongo客户端连接服务端
# 为了方便，也把二进制安装包中mongo客户端拷贝到本地mongoDB工作空间bin目录下
user@user-PC:~/Downloads/mongodb_simple$ cp ../mongodb-linux-x86_64-debian71-3.4.9/bin/mongo bin/
# 查看客户端使用说明
user@user-PC:~/Downloads/mongodb_simple$ ./bin/mongo --help
MongoDB shell version v3.4.9
usage: ./bin/mongo [options] [db address] [file names (ending in .js)]
# 启动客户端，连接test库
user@user-PC:~/Downloads/mongodb_simple$ ./bin/mongo 127.0.0.1:12345/test
2017-10-18T09:38:41.406+0000 I CONTROL  [initandlisten] 
> db.shutdownServer()　# db.shutdownServer()可以关闭mongoDB服务端
shutdown command only works with the admin database; try 'use admin'
> use admin　# 切换到admin库，这里提示shutdown命令只能在admin库下执行
switched to db admin
> db.shutdownServer()　# 再次执行关闭服务端命令
server should be down...
2017-10-18T10:04:27.448+0000 I NETWORK  [thread1] trying reconnect to 127.0.0.1:12345 (127.0.0.1) failed
2017-10-18T10:04:27.448+0000 W NETWORK  [thread1] Failed to connect to 127.0.0.1:12345, in(checking socket for error after poll), reason: Connection refused
2017-10-18T10:04:27.448+0000 I NETWORK  [thread1] reconnect 127.0.0.1:12345 (127.0.0.1) failed failed 
> ^C　# ctrl + c 退出当前客户端
bye
# 查看日志，服务端成功关闭
user@user-PC:~/Downloads/mongodb_simple$ tail -f log/mongod.log 
2017-10-18T10:04:27.180+0000 I NETWORK  [conn3] shutdown: going to close listening sockets...
2017-10-18T10:04:27.180+0000 I NETWORK  [conn3] closing listening socket: 7
2017-10-18T10:04:27.180+0000 I NETWORK  [conn3] closing listening socket: 8
2017-10-18T10:04:27.180+0000 I NETWORK  [conn3] removing socket file: /tmp/mongodb-12345.sock
2017-10-18T10:04:27.180+0000 I NETWORK  [conn3] shutdown: going to flush diaglog...
2017-10-18T10:04:27.180+0000 I FTDC     [conn3] Shutting down full-time diagnostic data capture
2017-10-18T10:04:27.182+0000 I STORAGE  [conn3] WiredTigerKVEngine shutting down
2017-10-18T10:04:27.445+0000 I STORAGE  [conn3] shutdown: removing fs lock...
2017-10-18T10:04:27.445+0000 I CONTROL  [conn3] now exiting
2017-10-18T10:04:27.445+0000 I CONTROL  [conn3] shutting down with code:0

```

## 关闭mongod服务端

* 如果中途有关闭数据库进程切记不要用`kiil -9`这样会造成锁死现象 用`user@user-PC:~/Downloads/mongodb_simple$ pkill mongod `命令比较好或者使用`kill -15` 或者不带任何参数或者在客户端执行`db.shutdownServer()`
* 如果无法启动建议删除data下面的mongod.lock,仅用于测试环境,记得删除之前备份

### 附录一:

mongoDB二进制安装包中bin文件夹下面文件作一个简单说明

|文件名     |作用              |
|-----------|------------------|
|mongod                          |mongoDB数据库服务端进程（数据库部署就是这个程序）|
|mongo                           |用于连接mongoDB数据库服务器的客户端|
|mongos                          |分片控制器|
|mongofiles                      |把从MongoDB GridFS获取文件实用程序|
|mongoimport                     |mongoDB数据的导入(JSON, CSV)|
|mongoexport                     |mongoDB数据的导出(JSON, CSV)|
|mongodump                       |以二进制的形式备份数据|
|mongorestore                    |从备份中恢复数据|
|mongooplog                      |操作日志的回放，复制集中用于记录操作记录的数据集合|
|mongostat                       |mongoDB服务器的各种状态，用户监控|



### 附录二:

mongod --help常用启动参数配置说明

|参数      　　　　　 　|作用              |默认值          |
|-----------------------|------------------|----------------|
|--dbpath   　　　　　　|数据文件的位置　　|/data/db/       |
|--fork     　　　　　　|是否后台执行　　　|默认前台执行    |
|--logpath  　　　　　　|日志文件的位置　　|输出到控制台    |
|--port     　　　　　　|端口　　　　　　　|27017           |
|-f [--config]       　 |制定配置文件　　　|　　            |
|-h [--help]        　　|帮助文档　　　　　|　　            |
|auth               　　|开启权限认证　　　|false           |

### 附录三:

mongod服务端启动配置文件mongod.conf

```properties
port = 12345
#可以使用相对路径也可以使用绝对路径，这里使用相对路径
dbpath = data
logpath = log/mongod.log
fork = true
```



  [1]: https://www.mongodb.com/