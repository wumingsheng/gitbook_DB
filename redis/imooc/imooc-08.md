# set数据结构

## redis的数据结构－set

### 概述

1. 和list不同的是，set集合中不允许出现重复的元素
2. set集合中，元素没有顺序＼
3. set可以包含最大元素数量是4294967295

### 使用场景
1. 跟踪一些唯一性数据
2. 维护对象之间的关系

### 常用命令

```java
//添加元素
jedis.sadd("user","sss","dddd");
//获取所有元素
Set<String> smembers = jedis.smembers("user");
//删除制定元素
jedis.srem("user", "sss");
//判断元素是否存在
Boolean sismember = jedis.sismember("user", "4566");
//差集运算（和顺序有关系）
jedis.sadd("users1","sss","dddd");
jedis.sadd("users2","sss","dddd","333");
Set<String> sdiff = jedis.sdiff("users2","users1");
//交集运算
Set<String> sinter = jedis.sinter("users2","users1");
//并集运算
Set<String> sunion = jedis.sunion("users2","users1");
//set中元素个数
jedis.scard("users1")
//从集合中弹出一个元素(由于集合是无序的，所有SPOP命令会从集合中随机选择一个元素弹出)，删除
String spop = jedis.spop("users1");
//从set中取一个随机元素，不删除元素
String srandmember = jedis.srandmember("users1");
//差集存到新的set中
Long sdiffstore = jedis.sdiffstore("users3", "users2","users1");
//交集存储
jedis.sinterstore("users3", "users2","users1");
//并集存储
jedis.sunionstore("users3", "users2","users1");
```


