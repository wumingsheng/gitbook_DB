# redis的持久化

## redis的持久化

两种持久化方式

- RDB方式（默认）
- AOF方式（配置）
- 无持久化（配置）
- 同事使用RDB和AOP(配置)

### RDB方式

默认支持，不需要配置；在制定的时间内，将数据集快照写入磁盘

优势：文件恢复，移植遍历，性能最大化
劣势：数据完整性低
配置：redis.conf配置文件
```shell
#900秒至少有一个key发生变化，保存一个快照
save 900 1
#每300秒至少有10个可以发生变化，持久化到文件一次
save 300 10
#60秒至少有10000个key发生变化，持久话一次
save 60 10000

# 文件保存路径
# The filename where to dump the DB
dbfilename dump.rdb


```

### AOF方式

以日志的形式，记录服务器处理的每一次操作，redis服务器启动之初，redis会读取日志文件，重新构造数据库数据

优势：高数据安全性（每次操作都同步数据），对日志append形式，如果日志过大，自动启动重写模式
劣势：效率较低（效率和安全性往往矛盾）
配置：redis.conf配置文件

```shell

# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.

#开启aof模式
appendonly yes

# The name of the append only file (default: "appendonly.aof")
# 文件名
appendfilename "appendonly.aof"

# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

#每个操作都同步
appendfsync always
#每妙同步
#appendfsync everysec
#不同步
# appendfsync no


```