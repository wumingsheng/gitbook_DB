# 【Chapter-3】Redis数据特性



- 位图（BitMap）:在某些情况下，使用位图带图字符串以节省内存空间
- 过期时间（expiration）:临时数据可以设置过期时间，以及键过期时发生的动作
- 排序（sorting）:redis支持对redis列表list、集合set、有序集合zset中的值进行排序并输出排序后的结果
- 管道（pipeline）:redis管道的使用，管道可以优化多个redis操作性能
- 事物（transaction）:redis支持类似关系数据库的事物
- 发布订阅（pubsub）:消息交换管道，消息队列
- 编写调试lua脚本:实现程序的可配置性和可扩展性，lua脚本可以将多个操作打包在一起并原子性执行



## 1. 使用位图（BitMap）

位图存储的是布尔信息，较字符串可以节省内存空间


位图是由比特位（bit）组成的数组，bit是用0/1标记的，所以位图只能存储布尔类型的信息0/1.实际上是设置指定数组索引(偏移量)的bit值0/1



案例：用户（ID）是否使用过某个功能（例如点赞）



```bash
#设置id为100用户已经点赞，标记为1
127.0.0.1:6379> setbit is_thumb_up 100 1
(integer) 0
#查询id为100用户是否点赞
127.0.0.1:6379> getbit is_thumb_up 100
(integer) 1
#统计点过赞的用户的个数
127.0.0.1:6379> bitcount is_thumb_up 
(integer) 1
#bitop命令用户进行位操作，该命令支持四种位操作：AND,OR,XOR,NOT
127.0.0.1:6379> setbit is_eat 100 1
(integer) 0
127.0.0.1:6379> setbit is_drink 100 1
(integer) 0
127.0.0.1:6379> bitop and is_eat_drink is_eat is_drink
(integer) 13
127.0.0.1:6379> getbit is_eat_drink 100
(integer) 1
```


总结：数据量大可以使用位图，数据量小使用集合

## 设置键的过期时间


```bash
127.0.0.1:6379> set name wms
OK
#expire命令:设置key的过期时间位30秒
127.0.0.1:6379> expire name 30
(integer) 1
#ttl命令：查看key的剩余存活时间
127.0.0.1:6379> ttl name
(integer) 24
#persist命令，清楚key的过期时间，时期成为持久的键，永不过期
127.0.0.1:6379> persist name
(integer) 1
```



我们可以通过以下三种方式清楚一个键的过期时间：

- 使用persist命令使其成为持久的键
- 键的值被替换或删除。包括set、getset和score命令会清楚过期时间。不过，修改列表，集合和哈希的元素不会清楚过期时间
- 被重命名为一个没有过期事假的key

如果键存在没有设置过期时间ttl返回-1；如果键不存在，返回-2；


## 使用sort命令


redis列表或者集合中的元素是无序的，有序集合中的元素是根据权重排序的，我们可以以某种非权重顺序对列表、集合、有序集合中的元素进行排序。

语法：
```
sort key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE destination]`
```


```bash
#对于纯数字而言，可以直接排序

127.0.0.1:6379> lpush age 12 34 54 2 22
(integer) 5
127.0.0.1:6379> sort age
1) "2"
2) "12"
3) "22"
4) "34"
5) "54"
#选项desc:降序
127.0.0.1:6379> sort age desc
1) "54"
2) "34"
3) "22"
4) "12"
5) "2"
#选项limit offset count 分页
127.0.0.1:6379> sort age limit 0 3 desc
1) "54"
2) "34"
3) "22"
#对于非数值的元素且想按照字段顺序对他们进行排序，那么我们需要增加修饰符ALPHA
127.0.0.1:6379> sadd name woms wms wumingsheng 122319
(integer) 4
127.0.0.1:6379> sort name alpha
1) "122319"
2) "wms"
3) "woms"
4) "wumingsheng"

```


## 使用管道（pipeline）


客户端将多个命令打包，并将他们一次性发送，而不再等待每个单独命令的执行结果，同时，redis管道需要服务器在执行所有的命令后再返回结果。



```bash
#安装工具unix2dos，可以将文本文件每一行都必须以\r\n结束
$ yum install unix2dos -y
#将命令写到一个文件中，每一行必须都以\r\n结束
$ cat > pipeline.txt << EOF
> set mykey myvalue
> sadd myset value1 value2
> get mykey
> scard myset
> EOF
$ unix2dos pipeline.txt 
unix2dos: converting file pipeline.txt to DOS format ...
#使用redis-cli的pipe选项，通过管道发送命令
$ cat pipeline.txt |bin/redis-cli --pipe
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 4
```


## 理解redis事物（transaction）

- multi:开启事物
- exec:执行事物
- discard:废弃事物

redis事物和关系性数据库事物之间的区别：redis事物没有事物回滚
  
  - 如果是语法错误，整个事物快速失败，所有命令不会被处理执行
  - 如果是在执行过程中发生错误，发生错误之后的命令仍然会执行，不会发生回滚



















