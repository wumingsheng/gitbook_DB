# redis实现分布式锁

## 1. 环境

```groovy

implementation 'org.springframework.boot:spring-boot-starter-data-redis'
implementation 'org.springframework.boot:spring-boot-starter-web'
compileOnly 'org.projectlombok:lombok'
```

```properties
spring.redis.host=127.0.0.1
spring.redis.port=6379
```

测试redis是否链接成功


我在redis里面有1000块钱
```

127.0.0.1:6379> keys *
1) "money:woms"
127.0.0.1:6379> get money:woms
"1000"

```

```java

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
	
	@Autowired
	private StringRedisTemplate stringRedisTemplate;
	
	@GetMapping("name")
	public Object name() {
		
		
		
		String womsMoney = stringRedisTemplate.opsForValue().get("money:woms");
		
		
		return womsMoney;
		
		
	}

}
```


```bash

$ curl localhost:8080/name -s|jq
1000

```

## 为什么要用分布式锁

当多个线程同时对value进行操作的话，就会有问题

模拟10个并发并发请求，每个请求的目的是扣除我的100快，这样1000块钱被10个请求执行，应该最后是0元，但是结果呢？

刚开始，我有1000
```

127.0.0.1:6379> set money:woms 1000
OK

```

10个线程同时消费，每个线程消费100

```java
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
	
	@Autowired
	private StringRedisTemplate stringRedisTemplate;
	
	@GetMapping("name")
	public Object name() throws Exception {
		
		for(int i=0; i<10; i++) {
			Thread thread = new Thread(() -> {
					
					Integer womsMoneyInteger = Integer.parseInt(stringRedisTemplate.opsForValue().get("money:woms"));
					stringRedisTemplate.opsForValue().set("money:woms", String.valueOf(womsMoneyInteger - 100));
					
			});
			thread.start();
		}
		
		TimeUnit.SECONDS.sleep(2);
		String now_money = stringRedisTemplate.opsForValue().get("money:woms");
		return now_money;
	}

}
```

最后看结果，发现我还有900

```
127.0.0.1:6379> get money:woms
"900"

```

为什么会出现这种情况呢？就是第一个用户get的时候，get的是1000，第二个用户get的时候，还是1000，第一个用户-100，剩900，第二个用户在之前的查询的1000的基础上-100，相当于应该-200，这里只-100

## redis实现分布式锁

```java

import java.util.concurrent.TimeUnit;

import org.springframework.data.redis.core.StringRedisTemplate;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class RedisLock {

	public boolean addLock(StringRedisTemplate redisTemplate, String key, String value, long timeout, int expire) {
		long nano = System.nanoTime();// 开始时间

		while ((System.nanoTime() - nano) < timeout) {
			if (redisTemplate.opsForValue().setIfAbsent(key, value, expire, TimeUnit.SECONDS)) {
				log.info(Thread.currentThread().getName());
				return true;
			}
		}
		return false;
	}

	public void unlock(StringRedisTemplate stringRedisTemplate, String key, String value) {

		String keyValue = stringRedisTemplate.opsForValue().get(key);
		if (value.equals(keyValue)) {
			log.info(Thread.currentThread().getName());
			stringRedisTemplate.delete(key);
		}

	}
}

```


```java

import java.util.UUID;
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.extern.slf4j.Slf4j;

@RestController
@Slf4j
public class TestController {

	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	@GetMapping("name")
	public Object name() throws Exception {
		

		for (int i = 0; i < 10; i++) {
			Thread thread = new Thread(() -> {
				String uuid = UUID.randomUUID().toString();
				RedisLock redisLock = new RedisLock();
				if (redisLock.addLock(stringRedisTemplate, "money:woms:lock", uuid, 1000 * 1000 * 1000L, 180)) {
					log.info(Thread.currentThread().getName());
					Integer womsMoneyInteger = Integer.parseInt(stringRedisTemplate.opsForValue().get("money:woms"));
					stringRedisTemplate.opsForValue().set("money:woms", String.valueOf(womsMoneyInteger - 100));
				} 
				redisLock.unlock(stringRedisTemplate, "money:woms:lock", uuid);
			});
			thread.start();
		}

		TimeUnit.SECONDS.sleep(10);
		String now_money = stringRedisTemplate.opsForValue().get("money:woms");
		return now_money;
	}

}

```

其原理就是使用另一个key作为当前操作的锁，给另一个key设置值，如果设置成功，可以对目标key的值进行操作，操作完成以后把作为锁的key删除。
其他用户由于给目标key设置值的时候，锁key值已近存在，就会在超时时间内一直设置值，如果超时时间内设置成功，则可以进行操作，设置不成功，就不进行操作，其目的就一个就是在同一个时间只能有一个用户对值进行修改。


如果使用lua脚本怎么实现呢？

```java

import java.util.Arrays;
import java.util.concurrent.TimeUnit;

import javax.annotation.PostConstruct;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.extern.slf4j.Slf4j;

@RestController
@Slf4j
public class TestController {

	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	private DefaultRedisScript<String> getRedisScript;

	@PostConstruct
	public void init() {
		getRedisScript = new DefaultRedisScript<String>();
		getRedisScript.setResultType(String.class);
		getRedisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("222.lua")));
	}

	@GetMapping("name")
	public Object name() throws Exception {

		for (int i = 0; i < 10; i++) {
			Thread thread = new Thread(() -> {
				log.info(Thread.currentThread().getName());
				stringRedisTemplate.execute(getRedisScript, Arrays.asList("money:woms"));

			});
			thread.start();
		}

		TimeUnit.SECONDS.sleep(10);
		String now_money = stringRedisTemplate.opsForValue().get("money:woms");
		return now_money;
	}

}


```

```lua
local money = redis.call("get",KEYS[1])
redis.call("set", KEYS[1], money - 100)
```


lua本身就是原子性，可以不用任何锁，直接操作











