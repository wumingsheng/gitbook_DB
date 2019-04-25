# 【第六章】SpringBoot整合MongoDB

# 第六章：springboot整合mongoDB

标签（空格分隔）： mongoDB

---

## 依赖包(org.springframework.boot:spring-boot-starter-data-mongodb)

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.8.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-data-mongodb')
	runtime('org.springframework.boot:spring-boot-devtools')
	compile('org.springframework.boot:spring-boot-starter-test')
}
```
## 配置文件(springboot自动配置)

```properties
spring.data.mongodb.host=localhost
spring.data.mongodb.username=imooc
spring.data.mongodb.password=123456
spring.data.mongodb.database=imooc
spring.data.mongodb.port=12345
```

## 增删改查

定义数据库对象
```java
public class Message {


    @Id
    private String id;

    private String title;


    private String content;

   // setter...
   // getter...
}

```
定义接口，MongoRepository里已近有crud操作，这里可以定义额外的接口
```java
import org.springframework.data.mongodb.repository.MongoRepository;

public interface MessageRepository extends MongoRepository<Message,String>{


    Message getMessageByTitle(String title);

}
```

测试类
```java

package com.example.demo.service;


import com.example.demo.DemoApplication;
import com.example.demo.mongo.Message;
import com.example.demo.mongo.MessageRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = DemoApplication.class)
public class MongoService {

    @Autowired
    private MessageRepository messageRepository;

    @Test
    public void demo(){


        System.out.print("hello world");
        System.out.println("=====================================================================");

        Message message = new Message();
        message.setContent("hello world");
        message.setContent("hw");
        message.setId("1");

        Message insert = messageRepository.insert(message);


        System.out.println("ok");

    }



}
```
启动类
```java

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

```