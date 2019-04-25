# string数据结构

## redis的数据结构-String

1. 字符串（String）
2. 字符串列表（list）
3. 哈希（hash）
4. 字符串集合（set）
5. 有序字符串集合（sorted set)


key定义的注意点：

- 不要太长
- 不过太短
- 同意命名规范

### 存储String

- 二进制存储，安全，存入和获取的数据相同
- value最多可以容纳的数据长度是512M

常用的命名：

```java
//赋值
jedis.set("demo", "dfjsdldf");
//取值
String value = jedis.get("demo");
//先获取旧值，再赋予新值
String value2 = jedis.getSet("demo", "laozhi");
//删除
Long del = jedis.del("demo");
//递增１　初始值默认０
Long incr = jedis.incr("id");
//递减１　默认初始值０
Long decr = jedis.decr("id");
//递增
Long incrBy = jedis.incrBy("id", 5);
//递减
Long incrBy = jedis.decrBy("id", 5);
//拼接字符串，返回字符串长度
Long append = jedis.append("id", "456");
```


