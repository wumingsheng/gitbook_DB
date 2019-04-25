# hash数据结构

## redis的数据结构－hash

- String Key和String　Value的map容器
- 每一个hash可以存储4294967295个键值对

### 常用命令

```java
//赋值，一次设置一个字段
jedis.hset("user", "name", "jack");
//取值，一个取一个字段
String hget = jedis.hget("user", "name");
//一次设置多个字段
Map<String,String> map = new HashMap<>();
map.put("name", "tom");
map.put("age", "12");
jedis.hmset("user",map );
//一次取多个字段
List<String> list = jedis.hmget("user", "name","age");
//一次取全部字段
Map<String, String> hgetAll = jedis.hgetAll("user");
//一次删除一个或者多个字段,返回删除个数
Long hdel = jedis.hdel("user","name","age");
//增加数字
jedis.hincrBy("user", "age", 3);
//查询字段是否存在，不存在赋值，存在不操作
Long hsetnx = jedis.hsetnx("user", "iphone", "18201182004");
//获取key的字段个数
Long hlen = jedis.hlen("user");
//获取所有的键[name, age]
Set<String> hkeys = jedis.hkeys("user");
//获取所有的值[12, tom]
List<String> hvals = jedis.hvals("user");
```


