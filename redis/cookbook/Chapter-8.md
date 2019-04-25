# 【Chapter-8】生产环境部署

# 生产环境部署



## 设置linux环境

1. 设置下列与内存相关的内核参数

```bash

[root@wms-redis-test-1 redis]# sysctl vm.overcommit_memory vm.swappiness
vm.overcommit_memory = 0
vm.swappiness = 30
[root@wms-redis-test-1 redis]# sysctl -w vm.overcommit_memory=1
vm.overcommit_memory = 1
[root@wms-redis-test-1 redis]# sysctl -w vm.swappiness=0
vm.swappiness = 0
[root@wms-redis-test-1 redis]# echo vm.overcommit_memory=1 >> /etc/sysctl.conf 
[root@wms-redis-test-1 redis]# echo vm.swappiness=0 >> /etc/sysctl.conf
[root@wms-redis-test-1 redis]# sysctl vm.overcommit_memory vm.swappiness       
vm.overcommit_memory = 1
vm.swappiness = 0

```



2. 禁用透明大页功能


```bash
[root@wms-redis-test-3 ~]# su -
Last login: Thu Oct 11 21:42:49 EDT 2018 from 172.19.0.197 on pts/0
[root@wms-redis-test-3 ~]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@wms-redis-test-3 ~]# echo never > /sys/kernel/mm/transparent_hugepage/defrag 
[root@wms-redis-test-3 ~]# cat >> /etc/rc.local << EOF
> echo never > /sys/kernel/mm/transparent_hugepage/enabled
> echo never > /sys/kernel/mm/transparent_hugepage/defrag
> EOF
[root@wms-redis-test-3 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled 
always madvise [never]
[root@wms-redis-test-3 ~]# cat /sys/kernel/mm/transparent_hugepage/defrag 
always madvise [never]
```

3. 网络参数优化

```bash
[root@wms-redis-test-3 ~]# sysctl net.core.somaxconn net.ipv4.tcp_max_syn_backlog
net.core.somaxconn = 128
net.ipv4.tcp_max_syn_backlog = 512
[root@wms-redis-test-3 ~]# sysctl -w net.core.somaxconn=65535
net.core.somaxconn = 65535
[root@wms-redis-test-3 ~]# sysctl -w net.ipv4.tcp_max_syn_backlog=65535
net.ipv4.tcp_max_syn_backlog = 65535
[root@wms-redis-test-3 ~]# echo "net.core.somaxconn=65535" >> /etc/sysctl.conf
[root@wms-redis-test-3 ~]# echo "net.ipv4.tcp_max_syn_backlog=65535" >> /etc/sysctl.conf
[root@wms-redis-test-3 ~]# sysctl net.core.somaxconn net.ipv4.tcp_max_syn_backlog       
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
```

4. 设置可以打开的最大文件数量

```bash
# 必须将nofile设置为一个小于如下命令的值
[root@wms-redis-test-3 ~]# cat /proc/sys/fs/file-max 
1610596
[root@wms-redis-test-3 ~]# su - redis
su: user redis does not exist
[root@wms-redis-test-3 ~]# ulimit -n 288000
# 使用如下命令查看结果
[root@wms-redis-test-3 ~]# ulimit -Hn -Sn
open files                      (-n) 288000
open files                      (-n) 288000
```



## redis安全

redis的安全非常有限，大部分依赖redis之外（操作系统，防火墙）的机制，redis的设计思想是它将部署在所有客户端都可信的环境中。
redis更侧重极限性能和简洁性，没有考虑支持完整的身份验证和访问控制。但是我们仍然可以做一些事情来保护redis服务器以免受非法的访问和攻击


## 配置客户端连接选项redis.conf



```

timeout 0 #默认
tcp-backlog 511 #默认
maxclients 10000 #默认
tcp-keepalive 300 #打开注释
client-output-buffer-limit normal 0 0 0  #默认
client-output-buffer-limit slave 256mb 256mb 60 #需要修改
client-output-buffer-limit pubsub 32mb 8mb 60 #默认

```


## 配置内存策略












