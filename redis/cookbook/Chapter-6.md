# 【Chapter-6】Redis持久化

# 持久化


为了保证数据的安全，redis还提供了将数据持久化到磁盘中的机制。redis提供了两种数据持久化类型：RDB和AOF

- RDB：可以看做是redis在某一个时间点上的快照，非常适合于备份和灾难恢复。默认开启
- AOF：是一个写入操作的日志，将在服务器启动时被重放。默认关闭

## 使用RDB

RDB默认是开启的，如果要禁用只需要注释掉配置文件中的save选项

```conf
save 900 1
save 300 10
save 60 10000
```

我们可以在redis的数据目录中查看是否生成了rdb文件dump.rdb

我们可以手动执行RDB快照，在redis-cli中调用save命令即可。redis-cli会被阻塞一段时间

```
[root@wms-redis-test-1 redis]# bin/redis-cli
127.0.0.1:6379> save
OK
```

或者我们可以执行bgsave命令来执行非阻塞的rdb转储，启动新进程转储

```
127.0.0.1:6379> bgsave
Background saving started
```

##RDB工作原理

RDB就像是一台专门给redis数据拍照的照相机，当满足触发策略时，redis就会通过将所有的数据转存到本地磁盘上一个文件的方式给redis的数据拍一张照片。
我们来解释一下save参数的含义，save参数决定了RDB触发策略，这个值的格式是x1,y1,x2,y2,x3,y3...
其含义是，如果超过y个键发生改变且此时没有转储正在发生，则在x秒后进行数据转储




可以同时使用多个策略来控制RDB快照的执行频率

```conf
save 900 1 #如果在900秒内有超过1个key发生了改变，则会进行一次RDB快照
save 300 10
save 60 10000
```


可以通过save或bgsave命令手动执行一次RDB快照，区别在于前者使用主线程同步转储，后者使用新线程异步转储

## RDB恢复

1. 要还原RDB快照，我们需要把快照文件（dump.rdb）复制到dir选项指定的位置，并保证启动redis实例的用户有该文件的读、写权限
2. 我们需要通过shutdown nosave 命令停止实例
3. 将新转储文件（要恢复的.rbd文件）重命名为dbfilename选项定义的名字，重新启动后，数据会从备份文件中加载并还原回redis了



## 使用AOF

RDB持久化并不能提供强的一致性
AOF（append-only file）是一种只记录redis写入命令的追加式日志文件。因为每个写入命令都会被追加到文件中，所以AOF的数据一致性比RDB更高。

AOF默认是关闭的，如果要开启AOF。可以在redis的配置文件中开启`appendonly yes`,然后重启reddis-server

使用`info persistence`判读是否启动了AOF功能，也可以检查数据目录是否生成了AOF文件


```bash
[root@wms-redis-test-1 redis]# ll
total 1724
-rw-r--r-- 1 root root      56 Oct  9 23:43 appendonly.aof #写入命令的追加文件
drwxr-xr-x 2 root root    4096 Oct  8 01:56 bin
drwxr-xr-x 2 root root    4096 Oct  9 23:42 conf
-rw-r--r-- 1 root root     109 Oct  9 23:44 dump.rdb #数据快照文件
drwxr-xr-x 2 root root    4096 Oct  9 22:59 log
drwxrwxr-x 6 root root    4096 Aug  3 18:44 redis-4.0.11
-rw-r--r-- 1 root root 1739656 Aug  3 18:46 redis-4.0.11.tar.gz
```

修复aof文件`bin/redis-check-aof --fix appendonly.aof`



## 同事使用RDB和AOF

- RDB默认开启
- AOF手动开启

```

save 900 1
save 300 10
save 60  10000
appendonly yes     # 只有这一项是需要改的
appendfilename "appendonly.aof"
appendfsync everysec
aof-use-rdb-preamble yes # AOF和rdb混合使用开关，需要手动改为yes，混合使用和同时使用是有区别的，混合使用可以节省开销同时发挥更好的优势
```



















