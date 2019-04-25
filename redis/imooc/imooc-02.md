# keys的通用操作

## keys的通用操作

```java
//所有keys
Set<String> keys = jedis.keys("*");
//以user开头的keys
Set<String> keys2 = jedis.keys("user?");
//删除key
jedis.del("user");
//确认一个key是否存在
jedis.exists("user");
//重命名
jedis.rename("old", "new");
//设置过期的时间，单位秒
jedis.expire("user", 1000);
//key剩余存活时间，没有设置存活时间返回-1
jedis.ttl("user");
//key对应的数据类型
jedis.type("user");
//删除当前所选数据库中所有key
jedis.flushDB();
//删除所有数据库中所有key
jedis.flushAll();
//当前数据库中key数目
jedis.dbSize();
//退出链接
jedis.quit();
//选择数据库
jedis.select(0);
```


