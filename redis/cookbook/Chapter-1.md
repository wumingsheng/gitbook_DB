# 【Chapter-1】开始使用Redis


## 下载和安装redis

redis官方网址：https://redis.io/

1. 安装编译工具
2. 为redis创建目录并切换到所创建的目录中
3. 下载redis
4. 解压到下载到redis源码并切换到对应的目录下
5. 为redis的配置文件创建目录，并把默认的配置文件复制进去
6. 编译依赖项
7. 编译redis
8. 安装redis
9. 进入/redis目录并验证生成了redis的二进制可执行文件

具体操作步骤如下


## 安装详细过程

```bash

[root@wms-redis-test-1 ~]# cd /
#为redis创建工作目录并切换到对应的目录下
[root@wms-redis-test-1 /]# mkdir redis
[root@wms-redis-test-1 /]# cd redis
[root@wms-redis-test-1 redis]# ll
total 0
# 下载redis最新版
[root@wms-redis-test-1 redis]# wget http://download.redis.io/releases/redis-4.0.11.tar.gz
-bash: wget: command not found
[root@wms-redis-test-1 redis]# yum -y install wget vim
[root@wms-redis-test-1 redis]# wget http://download.redis.io/releases/redis-4.0.11.tar.gz
# 解压并切换到对应的目录下
[root@wms-redis-test-1 redis]# tar -zxvf redis-4.0.11.tar.gz
[root@wms-redis-test-1 redis]# ll
total 1704
drwxrwxr-x 6 root root    4096 Aug  3 18:44 redis-4.0.11
-rw-r--r-- 1 root root 1739656 Aug  3 18:46 redis-4.0.11.tar.gz
[root@wms-redis-test-1 redis]# cd redis-4.0.11
[root@wms-redis-test-1 redis-4.0.11]# ll
total 312
-rw-rw-r--  1 root root 164219 Aug  3 18:44 00-RELEASENOTES
-rw-rw-r--  1 root root     53 Aug  3 18:44 BUGS
-rw-rw-r--  1 root root   1815 Aug  3 18:44 CONTRIBUTING
-rw-rw-r--  1 root root   1487 Aug  3 18:44 COPYING
drwxrwxr-x  6 root root   4096 Aug  3 18:44 deps
-rw-rw-r--  1 root root     11 Aug  3 18:44 INSTALL
-rw-rw-r--  1 root root    151 Aug  3 18:44 Makefile
-rw-rw-r--  1 root root   4223 Aug  3 18:44 MANIFESTO
-rw-rw-r--  1 root root  20543 Aug  3 18:44 README.md
-rw-rw-r--  1 root root  58766 Aug  3 18:44 redis.conf
-rwxrwxr-x  1 root root    271 Aug  3 18:44 runtest
-rwxrwxr-x  1 root root    280 Aug  3 18:44 runtest-cluster
-rwxrwxr-x  1 root root    281 Aug  3 18:44 runtest-sentinel
-rw-rw-r--  1 root root   7921 Aug  3 18:44 sentinel.conf
drwxrwxr-x  3 root root   4096 Aug  3 18:44 src
drwxrwxr-x 10 root root   4096 Aug  3 18:44 tests
drwxrwxr-x  8 root root   4096 Aug  3 18:44 utils
# 为redis配置文件创建目录，并把默认的配置文件复制进去
[root@wms-redis-test-1 redis-4.0.11]# mkdir /redis/conf
[root@wms-redis-test-1 redis-4.0.11]# cp redis.conf /redis/conf/
# 编译依赖项
[root@wms-redis-test-1 redis-4.0.11]# cd deps
[root@wms-redis-test-1 deps]# ll
total 28
drwxrwxr-x 4 root root 4096 Aug  3 18:44 hiredis
drwxrwxr-x 7 root root 4096 Aug  3 18:44 jemalloc
drwxrwxr-x 2 root root 4096 Aug  3 18:44 linenoise
drwxrwxr-x 6 root root 4096 Aug  3 18:44 lua
-rw-rw-r-- 1 root root 2518 Aug  3 18:44 Makefile
-rw-rw-r-- 1 root root 3517 Aug  3 18:44 README.md
-rwxrwxr-x 1 root root  282 Aug  3 18:44 update-jemalloc.sh
[root@wms-redis-test-1 deps]# make hiredis lua jemalloc linenoise
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
(echo "" > .make-cflags)
(echo "" > .make-ldflags)
MAKE hiredis
cd hiredis && make static
make[1]: Entering directory `/redis/redis-4.0.11/deps/hiredis'
gcc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  net.c
make[1]: gcc: Command not found
make[1]: *** [net.o] Error 127
make[1]: Leaving directory `/redis/redis-4.0.11/deps/hiredis'
make: *** [hiredis] Error 2
#编译报错，安装编译工具后继续编译
[root@wms-redis-test-1 deps]# yum groupinstall "Development Tools"
[root@wms-redis-test-1 deps]# make hiredis lua jemalloc linenoise
# 编译redis，看到如下提示，证明编译完成
[root@wms-redis-test-1 redis-4.0.11]# cd ..
[root@wms-redis-test-1 redis-4.0.11]# make   
Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory `/redis/redis-4.0.11/src'
# 安装redis
[root@wms-redis-test-1 redis-4.0.11]# make PREFIX=/redis install 
cd src && make install
make[1]: Entering directory `/redis/redis-4.0.11/src'

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/redis/redis-4.0.11/src'
# 进入redis目录并验证生成了redis的二进制可执行文件
[root@wms-redis-test-1 redis-4.0.11]# cd /redis/bin
[root@wms-redis-test-1 bin]# ll
total 21888
-rwxr-xr-x 1 root root 2451928 Oct  8 00:10 redis-benchmark
-rwxr-xr-x 1 root root 5776224 Oct  8 00:10 redis-check-aof
-rwxr-xr-x 1 root root 5776224 Oct  8 00:10 redis-check-rdb
-rwxr-xr-x 1 root root 2618016 Oct  8 00:10 redis-cli
lrwxrwxrwx 1 root root      12 Oct  8 00:10 redis-sentinel -> redis-server
-rwxr-xr-x 1 root root 5776224 Oct  8 00:10 redis-server




```


安装完成后，bin目录中会有一些可执行文件



|命令|说明|
|-----|-----|
|redis-benchmark：               |REDIS基准、性能测试工具|
|redis-check-aof：               |redis append only files(AOF)检查工具|
|redis-check-rdb：               |redis RDB检查工具|
|redis-cli：                     |redis命令行工具|
|redis-sentinel -> redis-server：|redis-server的软连接|
|redis-server：                  |redis服务端| 



## 启动和停止redis

1. 使用默认配置来启动一个redis实例

  $ bin/redis-server

2. 指定redis配置文件，例如使用从源码包拷贝过来的配置文件

  $ bin/redis-server conf/redis.conf

3. 如果是从操作系统的软件仓库中安装的redis，那么可以使用init.d脚本启动redis
  
  $ /etc/init.d/redis-server start

4. 如果要以redis-server守护进程的方式在后台启动redis，那么可以编辑配置文件并将daemonize参数设为yes，并使用该配置文件启动

  $ vim conf/redis.conf
    daemonize yes
  $ bin/redis-server conf/redis.conf

  Configuration loaded说明配置文件已经生效

5. 相应地，可以使用ctrl+c（如果redis是以前台模式启动的），或者kill+pid（如果redis是以后台模式启动的）来停止redis服务

  $ kill `pidof redis-server`

6. 更加优雅和推荐的停止redis的方式是通过redis-cli调用shutdown命令：

  $ cd /redis
  $ bin/redis-cli shutdown

7. 如果redis是从软件仓库中安装的话，那么还可以通过init.d脚本关闭

  $ /etc/init.d/redis-server stop





## 使用redis-cli连接到redis

打开终端，并通过redis-cli连接到redis

```bash

  [root@wms-redis-test-1 redis]# bin/redis-server conf/redis.conf 
  7182:C 08 Oct 02:11:45.425 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  7182:C 08 Oct 02:11:45.425 # Redis version=4.0.11, bits=64, commit=00000000, modified=0, pid=7182, just started
  7182:C 08 Oct 02:11:45.425 # Configuration loaded
  [root@wms-redis-test-1 redis]# bin/redis-cli
  127.0.0.1:6379>ping
  PONG
  127.0.0.1:6379> set name woms
  OK
  127.0.0.1:6379> get name
  "woms"
  127.0.0.1:6379> shutdown
  not connected> exit
  [root@wms-redis-test-1 redis]#
```


默认情况下，redis-cli会连接到127.0.0.1:6379（默认端口）上运行的redis实例，也可以使用

1. -h：指定主机名、ip  
2. -p：指定端口号
3. -a：指定密码（如果设置了连接密码）



## 获取服务器信息

1. 连接到一个redis实例，然后执行info命令

  $ bin/redis-cli
  127.0.0.1:6379> info
  
  结果如下
  ```
  # Server
  ...
  # Clients
  ...
  # Memory
  ...
  # Persistence
  ...
  # Stats
  ...
  # CPU
  ...
  Cluster
  cluster_enabled:0
  ```

2. 我们可以通过增加一个可选的<section>参数来指定获取那一部分信息


  ```
  127.0.0.1:6379> info server
  # Server
  redis_version:4.0.11
  redis_git_sha1:00000000
  redis_git_dirty:0
  redis_build_id:6b5c3692c9d6f3c0
  redis_mode:standalone
  os:Linux 3.10.0-693.el7.x86_64 x86_64
  arch_bits:64
  multiplexing_api:epoll
  atomicvar_api:atomic-builtin
  gcc_version:4.8.5
  process_id:7822
  run_id:f1cd71e4cc7520cc1c2ce09929a5f288df068632
  tcp_port:6379
  uptime_in_seconds:386
  uptime_in_days:0
  hz:10
  lru_clock:12253626
  executable:/redis/bin/redis-server
  config_file:/redis/conf/redis.conf
  ```

3. 使用redis-cli INFO命令，使用这种方式，可以方便的将命令的输出重定向到一个脚本中，进行指标分析和性能监控


  ```bash
  $ bin/redis-cli INFO
  $ bin/redis-cli INFO SERVER
  ```

4. INFO命令返回的信息

|名称|描述|
|-----|-----|
|Server     |关于redis服务器的基本信息|
|Clients    |客户端连接的状态和指标|
|Memory     |大致的内存消耗指标|
|Persistence|数据持久化相关的状态和指标|
|Stats      |总体统计数据|
|Replication|主从复制相关的状态和指标|
|CPU        |CPU使用情况|
|Cluster    |redis Cluster状态|
|Keyspace   |数据库相关的统计数据|


















