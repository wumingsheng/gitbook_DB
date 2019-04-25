# jedis入门

## jedis入门

- 直接获取链接

```java

//获取连接
Jedis jedis = new Jedis("172.18.115.44", 6380);

Set<String> keys = jedis.keys("*");

for (String key : keys) {
    
    System.out.println(key);
}
//释放资源
jedis.close();

```

- 使用连接池获取链接

```java

//使用连接池
JedisPoolConfig config = new JedisPoolConfig();
//设置最大连接数
config.setMaxTotal(30);
//设置最大空闲连接数
config.setMaxIdle(10);

JedisPool jedisPool = new JedisPool(config, "172.18.115.44", 6380);
Jedis jedis = jedisPool.getResource();

```

