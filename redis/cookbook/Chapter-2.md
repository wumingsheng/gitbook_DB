# 【Chapter-2】Redis数据类型


- 使用字符串（string）数据类型
- 使用列表 （list）数据类型，代号l
- 使用哈希 （hash）数据类型,代号h
- 使用集合（set）数据类型，代号s
- 使用有序集合（sorted set）数据类型，代号z
- 使用HyperLogLog数据类型，代号pf
- 使用Geo数据类型，代号geo
- 键管理


redis所有的key都是字符串类型

## string类型

特点:存储单一结构的数据

```bash
#查看所有的key
127.0.0.1:6379> keys *
1) "age"
2) "name"
#通过name获取value
127.0.0.1:6379> get name
"woms"
#查看value长度
127.0.0.1:6379> strlen name
(integer) 4
#如果name不存在返回nil
127.0.0.1:6379> get hello
(nil)
#append在末尾添加，如果name不存在，会创建一个空字符串，再去拼接
127.0.0.1:6379> append name 123
(integer) 7
127.0.0.1:6379> get name
"woms123"
127.0.0.1:6379> append hello world
(integer) 5
127.0.0.1:6379> get hello
"world"
#查询name是否存在，存在返回1不存在返回0
127.0.0.1:6379> exists name
(integer) 1
#不希望在key存在的时候，盲目的覆盖，使用setnx,如果key不存在，保存成功，返回1，如果key存在，返回0
127.0.0.1:6379> setnx name woms456
(integer) 0
127.0.0.1:6379> setnx addr beijing
(integer) 1
#删除当前数据库中所有的key，flushall删除所有数据库中的key
127.0.0.1:6379> flushdb
OK
127.0.0.1:6379> keys *
(empty list or set)
#mset/mget同时设置多个值，具有原子性
127.0.0.1:6379> mset name woms age 13 addr beijing
OK
127.0.0.1:6379> mget name age
1) "woms"
2) "13"
```
## 列表list类型

特点：有序可以重复列表

```bash
# 从左边插入两个值
127.0.0.1:6379> lpush name woms wms wumingsheng
(integer) 3
# 查询列表
127.0.0.1:6379> lrange name 0 -1
1) "wumingsheng"
2) "wms"
3) "woms"
# 从右端插入值
127.0.0.1:6379> lrange name 0 -1
1) "wumingsheng"
2) "wms"
3) "woms"
4) "122319"
#查看key的类型
127.0.0.1:6379> type name
list
#删除key
127.0.0.1:6379> del name
(integer) 1
# 在某一个值的前面、后面插入值
127.0.0.1:6379> lrange name 0 -1
1) "wumingsheng"
2) "wms"
3) "woms"
127.0.0.1:6379> linsert name after wms 122319
(integer) 4
127.0.0.1:6379> lrange name 0 -1
1) "wumingsheng"
2) "wms"
3) "122319"
4) "woms"
#查询指定索引位置的值，索引从0开始
127.0.0.1:6379> lindex name 2
"122319"
#lpushx、rpushx仅仅在列表存在的时候才插入值，key不存在，不插入值
127.0.0.1:6379> lpushx hello woms
(integer) 0
#lpop/rpop弹出一个元素
127.0.0.1:6379> lpop name
"bear"
#ltrim截取列表片段，通过索引，类似java中的string截取
127.0.0.1:6379> ltrim name 2 -1
OK
#lset设置指定索引位置的值
127.0.0.1:6379> lrange name 0 -1
1) "122319"
2) "woms"
127.0.0.1:6379> lset name 0 wms
OK
127.0.0.1:6379> lrange name 0 -1
1) "wms"
2) "woms"
```

## 哈希hash类型

特点：具有多属性的对象存储

```bash
#hmset设置多个字段-值，hset设置单个字段-值
127.0.0.1:6379> hmset wms name wumingsheng age 13 address beijing
OK
#hmget同时获取多个字段，hget获取一个字段
127.0.0.1:6379> hmget wms name age address
1) "wumingsheng"
2) "13"
3) "beijing"
127.0.0.1:6379> hget wms name
"wumingsheng"
#hexists查询key中是否存在某个字段
127.0.0.1:6379> hexists wms name
(integer) 1
#查询所有字段
127.0.0.1:6379> hgetall wms
1) "name"
2) "wumingsheng"
3) "age"
4) "13"
5) "address"
6) "beijing"
#删除某个字段
127.0.0.1:6379> hdel wms name
(integer) 1
```

hset和hmset会覆盖现在字段，hsetnx只有在字段不存在的时候才会设置字段的值

## 集合set类型

特点：无须，不可以重复

```bash
127.0.0.1:6379> keys *
(empty list or set)
#添加元素
127.0.0.1:6379> sadd name woms wms wumingsheng woms
(integer) 3
#查询元素的个数
127.0.0.1:6379> scard name
(integer) 3
#查看所有的元素
127.0.0.1:6379> smembers name
1) "woms"
2) "wumingsheng"
3) "wms"
#判断元素是否存在
127.0.0.1:6379> sismember name wms
(integer) 1
#删除元素
127.0.0.1:6379> srem name wms
(integer) 1
127.0.0.1:6379> sismember name wms
(integer) 0

```


redis提供了一组集合运算的相关命令

- SUNION和SUNIONSTORE用于计算并集
- SINTER和SINTERSTORE用于计算交集
- SDIFF和SDIFFSTORE用于计算差集

不带STORE后缀的命令只返回相应操作的结果集合，带STROE后缀的命令则会将结果存储到一个指定的键中

## 有序集合（sorted set）类型

特点：不可以重复，带有权重分数所以有序

- list：双向链条结构，有序可重复
- set: 集合结构，无序不可以重复
- zset: 集合结构所以不可以重复，有权重分数可以认为有序

```bash
#zadd添加元素
127.0.0.1:6379> zadd name 100 woms 90 wms 80 wumingsheng 70 woms
(integer) 3
#zrange和zrevrange命令获取排名
127.0.0.1:6379> zrevrange name 0 -1
1) "wms"
2) "wumingsheng"
3) "woms"
127.0.0.1:6379> zrevrange name 0 -1 withscores
1) "wms"
2) "90"
3) "wumingsheng"
4) "80"
5) "woms"
6) "70"
127.0.0.1:6379> zrange name 0 -1 withscores
1) "woms"
2) "70"
3) "wumingsheng"
4) "80"
5) "wms"
6) "90"
#zincrby key increment member 给集合元素添加权重分数
127.0.0.1:6379> zincrby name 5 woms
"75"
#zrank、zrevrank命令查看元素排名，zscore命令查看元素分数、权重
127.0.0.1:6379> zrank name woms
(integer) 0
127.0.0.1:6379> zrevrank name woms
(integer) 2
127.0.0.1:6379> zscore name woms
"75"

#ZUNIONSTORE命令用来合并两个集合






```

ZADD命令中使用NX选项，能够实现在不更新已存在的成员的情况下值添加新的成员
选项XX只更新存在的成员不添加新的元素
如果多个成员具有相同的分数权重，redis将按照字段顺序进行排序




## HyperLogLog类型

特点：集合唯一计数

当数据量增大到上千万时候，考虑到内存消耗和性能下降问题，如果我们不需要获取数据集的内容，只是想得到不同值的个数，可以使用HyperLogLog（HLL）
优点：大集合唯一计数，消耗内存小，耗时短
缺点：可能不准确，标准差小于1%
适合数据量大，精确度要求低

```bash
# pfadd命令用来添加元素
127.0.0.1:6379> pfadd name woms
(integer) 1
127.0.0.1:6379> pfadd name wms wumingsheng
(integer) 1
#pfcount命令用来统计元素的个数
127.0.0.1:6379> pfcount name
(integer) 3
127.0.0.1:6379> pfadd name woms
(integer) 0
127.0.0.1:6379> pfcount name
(integer) 3
127.0.0.1:6379> pfadd name2 woms
(integer) 1
#pfmerge元素用来合并元素，排除重复的元素
127.0.0.1:6379> pfmerge nameall name name2
OK
127.0.0.1:6379> pfcount nameall
(integer) 3

```


## GEO类型


特点：存储和查询地理位置坐标

```bash
#geoadd命令添加地理位置信息
127.0.0.1:6379> geoadd addr -121.896321 37.916750 "olive garden" -117.910937 33.804047 "P.F Changes" -118.508020 34.453276 "outback sthouse" -119.152439 34.264558 "Red lobster" -122.276909 39.458300 "longhorn charcoal pit"
(integer) 5
#geopos命令查询地理位置信息
127.0.0.1:6379> geopos addr "olive garden"
1) 1) "-121.89632266759872437"
   2) "37.91675061080587028"
#georadius命令查询方圆5公里内的点
127.0.0.1:6379> georadius addr -121.923170 37.878506 5 km
1) "olive garden"
#georadiusbymember命令和georadius命令非常相似，都可以用来找出指定范围内的成员，但是georadiusbymember命令的中心点是geo集合中的一个成员，georadius是使用输入的经纬度来决定中心点的位置
127.0.0.1:6379> georadiusbymember addr "outback sthouse" 100 km
1) "Red lobster"
2) "outback sthouse"
3) "P.F Changes"
#geodist查询两点之间的距离
127.0.0.1:6379> geodist addr "P.F Changes" "outback sthouse" km
"90.7557"

```
GEORADIUS和GEORADIUSBYMEMBER命令中，

- 使用WITHDIST选项来得到距离
- 使用ASC/DESC选项来控制返回结果的升序或降序
- 使用STORE/STOREDIST选项还可以将结果存储到另一个GEO集合中

GEO集合实际上被存储到一个有序集合（ZSET）,因此有序集合支持的所有命令可以用于GEO数据类型，
我们可以使用ZREM从集合中移除元素，也可以使用ZRANGE来获取所有的成员



## 键管理


```bash
#查询redis中key的个数
127.0.0.1:6379> dbsize
(integer) 0
#列出所有的key，可以使用keys命令，也可以使用scan命令
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> scan 0
1) "0"
2) (empty list or set)
#删除一个key，使用del命令，也可以使用unlink命令
127.0.0.1:6379> del name
(integer) 1
127.0.0.1:6379> unlink name
(integer) 1
#判断一个key是否存在
127.0.0.1:6379> exists name
(integer) 0
#判断一个key类型
127.0.0.1:6379> type name
string
#重命名一个key
127.0.0.1:6379> rename name name2
OK
#flushdb命令：删除当前数据库的所有key；
#flushall命令：删除所有数据库的所有key；
```









