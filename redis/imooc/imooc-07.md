# list数据结构

## redis的数据结构－list

### 概述

列表可以存储一个有序的字符串列表，常用信息队列场景

### 常用命令

```java
//左边添加元素
jedis.lpush("user","a","b","c");
//右边添加元素
jedis.rpush("user", "1","2","3");
//key存在插入元素，key不存在，不插入
jedis.lpushx("user3", "ddfs");
//查看列表[c, b, a, 1, 2, 3]索引从0开始-1代表最后一个元素
List<String> lrange = jedis.lrange("user", 0, -1);
//列表长度
jedis.llen("user");
//列表两端弹出元素-左弹-同时从列表中删除元素
String lpop = jedis.lpop("user");
//列表两端弹出元素-右弹-同时从列表中删除元素
String rpop = jedis.rpop("user");
//LREM命令会删除列表中前count个值为value的元素，返回实际删除的元素个数。
//根据count值的不同，该命令的执行方式会有所不同： 
//当count>0时， LREM会从列表左边开始删除。 当count<0时， LREM会从列表后边开始删除。 当count=0时， LREM删除所有值为value的元素
Long lrem = jedis.lrem("user", 0, "1");
//修改制定索引位置的值
jedis.lset("user", 1, "tomcat");
//获取制定索引位置的值
String lindex = jedis.lindex("user", 2);
//保留截取的列表
String ltrim = jedis.ltrim("user", 2, 3);
//指定的位置插入元素
jedis.linsert("user", LIST_POSITION.BEFORE, "b", "ddddd");
//将一个元素从一个列表尾部转到另一个列表头部
jedis.rpoplpush("user1", "user2");
```