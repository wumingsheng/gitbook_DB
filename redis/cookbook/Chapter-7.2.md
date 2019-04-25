# 【Chapter-7.2】配置高可用和集群

# REDIS-CLUSTER


主从模式保证了数据的备份，加上sentinel可以保证redis高可用，但是这只是数据的纵向扩展
为了让redis有更庞大的内存，容纳更多的数据，我们需要横向扩展redis，分布式cluster模式，数据分散存储在不同的redis上，减少一台redis内存不足带来的压力

- 横向扩展-主从（sentinel）-备份数据，保证数据的安全不丢失
- 纵向扩展-分布式（cluster）- 分布式（分区）存储数据，增大数据的存储能力

![](/redis/cookbook/img/5589.png)
![](/redis/cookbook/img/5588.png)




使用redis-trib.rb安装

1. 分别启动6个redis，

需要关注的redis配置文件redis.conf选项是
```
#后台方式运行
daemonize yes
logfile "/redis/log/redis.log"
#解决网络问题
bind 0.0.0.0 #或者本机的ip
cluster-enabled yes # 开启集群模式
```

以下配置选项可供参考

```
port 6379 # redis服务端口
daemonize yes #守护进程方式后台运行redis
bind 172.18.113.66 # 本机的ip地址local-ip，也可以设置为0.0.0.0
appendonly yes #启动aof
aof-use-rdb-preamble yes #aof和rdb混合模式
appendfilename "appendonly_6379.aof" # aof持久化写命令文件
pidfile /var/run/redis_6379.pid # 进程文件 
logfile "/redis/log/redis_6379.log" # 日志输出文件
dir ./ #工作目录
dbfilename dump_6379.rdb # rdb持久化快照文件
cluster-enabled yes # 开启集群模式
cluster-config-file nodes-6379.conf # 集群信息文件
cluster-node-timeout 15000 # 集群超时时间
```

2. 执行脚本


```bash
# 安装ruby环境,新版ruby会自带gem，老版本需要额外安装yum -y install rubygems
yum -y install ruby
# 检查是否安装成功
ruby -v
gem -v
# 安装ruby和redis的接口程序,如果报错版本错误，https://blog.csdn.net/qq_37595946/article/details/77800147
gem install redis 
# 从源码中复制脚本出来
cp redis-4.0.11/src/redis-trib.rb /redis
# 执行脚本
ruby redis-trib.rb create --replicas 1 172.18.113.63:6379 172.18.113.63:6380 172.18.113.66:6379 172.18.113.66:6380 172.18.113.65:6379 172.18.113.65:6380   

```


3. 测试redis cluster

可以随意登录到一个节点

```bash
# -c 表示以cluster方式连接redis，使用命令cluster info和cluster node查看集群状态
$ bin/redis-cli -c -h 172.18.113.66 -p 6379 
172.18.113.66:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_ping_sent:874
cluster_stats_messages_pong_sent:848
cluster_stats_messages_meet_sent:4
cluster_stats_messages_sent:1726
cluster_stats_messages_ping_received:847
cluster_stats_messages_pong_received:878
cluster_stats_messages_meet_received:1
cluster_stats_messages_received:1726
172.18.113.66:6379> cluster nodes
3637a13f9d040dbc93568a833798d0b7fbadbea5 172.18.113.63:6379@16379 master - 0 1539239832353 1 connected 0-5460
c50780389596ecb671410b1bdd2aadc4f7dc8e74 172.18.113.63:6380@16380 slave 3aebc3e116bf5332124daad622d005dd13efb63b 0 1539239830351 5 connected
40c69be93e2319787e9a01ecd140c3178f93cacf 172.18.113.66:6380@16380 slave 3637a13f9d040dbc93568a833798d0b7fbadbea5 0 1539239829349 4 connected
3aebc3e116bf5332124daad622d005dd13efb63b 172.18.113.65:6379@16379 master - 0 1539239831352 5 connected 10923-16383
1cffb7b58e2e8d07286db854e81e3e01c3af3dfd 172.18.113.65:6380@16380 slave 89002a5f07adadf40c65f2c93ecee9e0e6659520 0 1539239830000 6 connected
89002a5f07adadf40c65f2c93ecee9e0e6659520 172.18.113.66:6379@16379 myself,master - 0 1539239829000 3 connected 5461-10922
172.18.113.66:6379> 
```



也可以使用脚本检查，可以随意指定一个节点
```bash
# ruby redis-trib.rb check 172.18.113.66:6379
>>> Performing Cluster Check (using node 172.18.113.66:6379)
M: 89002a5f07adadf40c65f2c93ecee9e0e6659520 172.18.113.66:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 3637a13f9d040dbc93568a833798d0b7fbadbea5 172.18.113.63:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: c50780389596ecb671410b1bdd2aadc4f7dc8e74 172.18.113.63:6380
   slots: (0 slots) slave
   replicates 3aebc3e116bf5332124daad622d005dd13efb63b
S: 40c69be93e2319787e9a01ecd140c3178f93cacf 172.18.113.66:6380
   slots: (0 slots) slave
   replicates 3637a13f9d040dbc93568a833798d0b7fbadbea5
M: 3aebc3e116bf5332124daad622d005dd13efb63b 172.18.113.65:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 1cffb7b58e2e8d07286db854e81e3e01c3af3dfd 172.18.113.65:6380
   slots: (0 slots) slave
   replicates 89002a5f07adadf40c65f2c93ecee9e0e6659520
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



4. 管理redis-cluster

 - 获取集群的运行拓扑和状态
 - 轻松的添加或删除节点



获取集群的状态
bin/redis-cli -h 172.18.113.66 -p 6379 -c cluster info
获取集群中节点的状态
bin/redis-cli -h 172.18.113.66 -p 6379 -c cluster nodes
手动触发故障，让一个实例提升为主实例
bin/redis-cli -h 172.18.113.66 -p 6379 -c cluster failover
从指定的主实例中，获取从实例的信息
bin/redis-cli -h 172.18.113.65 -p 6379 -c cluster slaves 1cffb7b58e2e8d07286db854e81e3e01c3af3dfd



向正在运行的集群添加一个分片（主实例和他的从实例）
  - 准备和启动两个实例（配置选项参考1小节的配置选项，也就是和创建集群之前准备的实例相同即可）
  - 添加主实例和从实例








  ```bash
  # 6381是被添加的实例节点 6379是原来集群的任意一个节点
  ruby redis-trib.rb add-node 172.18.113.66:6381 172.18.113.66:6379
  # 用脚本测试
  ruby redis-trib.rb check 172.18.113.66:6381
  # 可以看到4个master节点，三个slave节点，刚加入的节点变成了master节点，但是并没有分配哈希卡槽（0 slots）
  # redis-cluster在新增节点时并未分配卡槽，需要我们手动对集群进行重新分片迁移数据，需要重新分片命令 reshard
  ruby redis-trib.rb reshard 172.18.113.66:6379 #给集群重新分片，指定集群的任意一个节点都可以
  # 这个命令是用来迁移slot节点的，后面的节点是表示是哪个集群，任意一个节点都可以
  # 它提示我们需要迁移多少slot到新master节点上，我们平分16384个哈希槽给4个节点：16384/4 = 4096，我们需要移动4096个槽点到新master上
  redis-trib 会向你询问重新分片的源节点（source node），即，要从特点的哪个节点中取出 4096 个哈希槽，还是从全部节点提取4096个哈希槽， 并将这些槽移动到新master节点上面。





  #添加从节点6382，使用add-node --slave命令。
  ruby redis-trib.rb check 172.18.113.66:6379 #查看主节点的node-id，可以随意指定集群中任意一个节点
  ruby redis-trib.rb add-node --slave --master-id a4452d66662013a229adc0fdbdfa890fd0017471 172.18.113.66:6382 172.18.113.66:6379




  # 删除一个节点
  #redis-trib del-node 192.168.33.13:7009 `<node-id>`
  #如果是从节点可以直接删除，如果是主节点，如果有分配slot，需要先迁移
  # 先把这个主节点分配的slots删除了
  bin/redis-cli -h 172.18.113.66 -p 6381 -c cluster flushslots      
  ruby redis-trib.rb check 172.18.113.66:6379
  ruby redis-trib.rb del-node 172.18.113.66:6379 220c7c06b541e6deef03bf2be3d88dedfe5e9afa
  #手动分配slots 
  for i in {5461..5797};do bin/redis-cli -h 172.18.113.65 -p 6380 cluster addslots $i;done



  ```

参考：
- https://blog.csdn.net/think2me/article/details/50384924
- https://www.cnblogs.com/crazylqy/p/7456049.html















