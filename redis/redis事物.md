# redis事物

## redis只支持部分事物

redis不像关系型数据库，支持ACID，只是支持部分事物

这里的部分事物怎么理解？

redis没有事物回滚这么一说，因为没有事物回滚，每条命令没有正真执行，因此也获取不到执行结果

只是把所有的命令放在一个管道里，**最后顺序一起** 执行，将结果值一起返回

这里要解释成3点：

1. multi开始事物，事物期间的所有命令不执行，只是放在一个管道里面，命令根本没有执行，也获取不到执行结果，执行结果都是null
2. exec提交事物，管道里面的命令一起提交，顺序执行，能成功的就成功，不能成功的就失败，最后将结果一起返回
3. discard放弃命令，放弃管道里面的命令不去执行，所谓的同时成功同时失败，就是靠这三个逻辑命令控制的



![](/assets/20190414194019.png)


## 代码实例一：exec之前获取不到结果值，所有的结果值都是null,exec的时候一起返回结果值

```java
	stringRedisTemplate.setEnableTransactionSupport(true);//redis session支持事物
	stringRedisTemplate.multi();//开始事物
	stringRedisTemplate.opsForValue().set("key1", "value1", Duration.ofDays(1L));
	stringRedisTemplate.opsForValue().set("key2", "value2", Duration.ofDays(1L));
	stringRedisTemplate.opsForValue().set("key3", "value3", Duration.ofDays(1L));

	String string1 = stringRedisTemplate.opsForValue().get("key1");
	String string2 = stringRedisTemplate.opsForValue().get("key2");
	String string3 = stringRedisTemplate.opsForValue().get("key3");
	
	log.info("string1={},string2={},string3={}", string1, string2, string3);//string1=null,string2=null,string3=null
	List<Object> list = stringRedisTemplate.exec();//提交事物list=[true, true, true, value1, value2, value3]
	log.info("list={}", list);
		

```

## 代码实例二：exec之前有错误，会放弃所有的命令，做到`一起成功、一起失败`

```java

	log.info("say hello world ...");

		List<Object> exec = stringRedisTemplate.execute(new SessionCallback<List<Object>>() {
		

			@Override
			public List<Object> execute(RedisOperations operations) throws DataAccessException {
				
					stringRedisTemplate.setEnableTransactionSupport(true);
					operations.multi();// 开启事物
					operations.opsForValue().set("key1", "value1", Duration.ofDays(1L));
					operations.opsForValue().set("key2", "value2", Duration.ofDays(1L));
					operations.opsForValue().set("key2", null, Duration.ofDays(1L));//这里有问题
					return operations.exec();
					
				
			}

		});

		log.info("exec={}", exec);

		return exec;


```

## 代码实例三：exec中发生错误，不能做到`一起成功、一起失败`，成功的就成功了，失败的就失败了

```java


log.info("say hello world ...");

		List<Object> exec = stringRedisTemplate.execute(new SessionCallback<List<Object>>() {
		

			@Override
			public List<Object> execute(RedisOperations operations) throws DataAccessException {
				
					stringRedisTemplate.setEnableTransactionSupport(true);
					operations.multi();// 开启事物
					operations.opsForValue().set("key1", "value1", Duration.ofDays(1L));//没问题的就执行了
					operations.opsForValue().set("key2", "value2", Duration.ofDays(1L));
					operations.opsForSet().add("key1", "aaa", "vvv");//这一行报错了，就不执行了
					operations.opsForValue().set("key3", "value3", Duration.ofDays(1L));
					return operations.exec();
				
			}

		});

		log.info("exec={}", exec);

```

> 虽然代码会报错抛出异常，但是管道里面的命令，能成功的就已近成功执行了


