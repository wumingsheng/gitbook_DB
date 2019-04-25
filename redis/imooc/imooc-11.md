# redis的特性

## redis的特性

### redis支持多数据库

一个redis最多支持16个数据库，下标从0到15,默认连接0数据库，也可以通过select选择具体链接那个数据库

```java
//选择１号数据库
String select = jedis.select(1);
//移动key到制定的数据库
jedis.move("a", 1);
```

### 事物支持

- multi开启事物begin
- exec提交事物commit
- discard回滚事物rollback
		
```java
//multi开启事物begin
Transaction multi = jedis.multi();
multi.set("user", "1");
multi.incr("user");
//exec提交事物commit
//	multi.exec();
//discard回滚事物rollback
multi.discard();
```

