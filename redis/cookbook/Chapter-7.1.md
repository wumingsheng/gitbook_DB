# 【Chapter-7.1】配置高可用和集群

# 配置高可用和集群


使用redis的主从复制和持久化可以解决数据备份的问题。但是，当主机宕机时，整个redis服务将无法恢复。
redis2.6以后，原生支持的sentinel是使用最广泛的高可用架构。
利用sentinel，我们可以轻松的构建具备容错能力的redis服务。


-----

## 配置sentinel

之前提高的redis主从机制，当主节点下线，从节点是不会接替主节点的角色。如果主节点宕机了，就无法保证高可用；
sentinel就是保证如果master宕机了，slave接替master继续工作
sentinel（哨兵）充当了redis主实例和从实例的守卫者，因为单个哨兵本身也可能失效。所以一个哨兵显然不足以保证高可用。
对主实例进行故障迁移的策略是基于仲裁系统的，所以至少需要3个哨兵进程才能构成一个健壮的分布式系统来持续的监控redis主实例的状态。
如果有多个哨兵进程检测到主实例下线，其中的一个哨兵进程会被选举出来负责推选一个从实例代替原有的主实例。

演示如何配置一个由三个哨兵监控的1主2从环境



|角色          |     ip地址         |     端口号    |
|-----         |     -----          |     -----     |
| master       |     172.18.113.66  |       6379    |
| slave-1      |     172.18.113.63  |       6379    |
| slave-2      |     172.18.113.65  |       6379    |
| sentinel-1   |     172.18.113.66  |       26379   |
| sentinel-2   |     172.18.113.63  |       26379   |
| sentinel-3   |     172.18.113.65  |       26379   |




1. 下载编译安装

```bash
cd /redis #切换到redis的安装工作目录
mkdir conf #创建配置文件目录
mkdir log #创建日志输出文件目录
cp redis-4.0.11/redis.conf conf/ #拷贝一份redis的配置文件
cp redis-4.0.11/sentinel.conf conf/ #拷贝一份sentinel的配置文件
```




2. 修改配置文件



  redis.conf
  ```

  bind 0.0.0.0 #网络
  daemonize yes #守护进程
  logfile "/redis/log/redis.log" #日志输出文件
  appendonly yes #开启aof持久化方式
  aof-use-rdb-preamble yes #开启aof和rdb混合使用模式
  ```



sentinel.conf配置项如下
所有的sentinel节点配置项都相同
```

bind 0.0.0.0 #解决网络问题 也可以是bind 127.0.0.1 localhost-ip
daemonize yes  #这个是手动加的，模板配置文件中没有，以守护进程的方式启动
logfile "/redis/log/sentinel.log" #这个也是手动加的，指定日志输出文件
sentinel monitor mymaster 172.18.113.66 6379 2 #这个地方也是相同的，指定的都是初始master-ip

```

3. 分别复制两份redis给两个slave节点`scp -r /redis root@172.18.113.63:/`

4. 修改slave节点区别于master的配置


```
slaveof 172.18.113.66 6379 #slave节点配置，master节点不配置，所有的slave节点配置项都相同，都是指向master的ip、port
```






5. 分别登录到master节点和slave节点，启动redis-server`bin/redis-server conf/redis.conf` 


6. 启动sentinel

备注：sentinel进程的启动方式有两种：两种方式效果相同

- bin/redis-sentinel conf/sentinel.conf
- bin/redis-server conf/sentinel.conf --sentinel




7. 查看replication信息

bin/redis-cli -h 172.18.113.66 -p 6379 info replication
bin/redis-cli -h 172.18.113.63 -p 6379 info replication
bin/redis-cli -h 172.18.113.65 -p 6379 info replication



```

# Replication
role:master
connected_slaves:2
slave0:ip=172.18.113.63,port=6379,state=online,offset=28216,lag=1
slave1:ip=172.18.113.65,port=6379,state=online,offset=28357,lag=0

bin/redis-cli -h 172.18.113.63 info replication 
# Replication
role:slave
master_host:172.18.113.66
master_port:6379
master_link_status:up


bin/redis-cli -h 172.18.113.65 info replication 
# Replication
role:slave
master_host:172.18.113.66
master_port:6379
master_link_status:up

```
8. 测试关闭master节点，看master是否会转移

  - bin/redis-cli -h 172.18.113.66 shutdown
  - bin/redis-cli -h 172.18.113.63 info replication 
  - bin/redis-cli -h 172.18.113.65 info replication 

备注：master切换需要几秒钟的时间

9. 关闭redis，关闭sentinel




- bin/redis-cli -h 172.18.113.66 -p 6379 shutdown
- bin/redis-cli -h 172.18.113.66 -p 26379 shutdown   #26379是sentinel的端口号

备注：默认 -h 127.0.0.1 -p 6379



























