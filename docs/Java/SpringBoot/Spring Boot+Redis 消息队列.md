# Spring Boot+Redis 消息队列

[toc]



## 什么是消息队列

消息队列中有三个角色：生产者、消费者、队列。

生产者负责创建消息并放入队列中；消费者负责从队列中取出并处理消息。

如果队列为空，消费者阻塞；如果队列为满，生产者阻塞。

使用消息队列的好处是“削峰”，也即当大量请求涌入时，可以先将请求放到队列中，然后慢慢处理。



## 设计思路

1. 实现步骤：

   1. 开启Spring Boot的异步执行机制。

   2. 创建线程池，并初始化线程池中基于内存的消息队列。

      【注】上述两步可以参考我写的一篇文章《Sping Boot 异步事件，线程池，任务队列》。
      
   3. 基于“生产者-消费者”模型，在项目刚启动的时候为消费者分配一个线程并一直执行（`while (true)`死循环）。当任务队列为空时，让消费者阻塞；当有任务时，每次执行一个任务。

2. 为什么不直接使用基于内存的消息队列呢？

   答：其实在最开始的版本中，我采用的就是线程池+消息队列的方式实现“削峰”。但是缺点是我没法方便的查看当前队列中有哪些消息。而将消息放在Redis中，即可通过Redis客户端方便查看队列中有哪些消息。



## 详细步骤

### 一）Spring Boot 整合 Redis

参考我的另一篇文章**《springboot整合redis》**。



### 二）案例代码

#### Maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--springboot redis-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <!--json工具-->
    <dependency>
        <groupId>com.alibaba.fastjson2</groupId>
        <artifactId>fastjson2</artifactId>
        <version>2.0.22</version>
    </dependency>

    <!--lombok注解-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```



#### 设置配置文件`application.properties`

注意：Spring Boot 2.X 和 3.X 的写法有一点不同。

```properties
spring.redis.database=0
spring.redis.host=192.168.248.128
spring.redis.port=6379
spring.redis.password=Redis数据库密码
# springboot 2.x 中，一定要设置一个大于0的数。如果一次操作Redis的时间超过这个数，则会报错。
spring.redis.timeout=1000000000000
```



#### Redis序列化配置类

```java
package com.lee.redismq.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;

/**
 * @author 晓龙coding
 */
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(redisConnectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 设置key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        return template;
    }
}
```



#### 测试Spring Boot项目是否连接上Redis服务器

```java
@SpringBootTest
class RedismqApplicationTests {
    @Resource
    private RedisTemplate<String, Object> redisTemplate;
    @Test
    void contextLoads() {
        redisTemplate.opsForValue().set("name", "ikun");
        System.out.println("name=" + redisTemplate.opsForValue().get("name"));
    }
}
```



#### 开启异步机制

具体来说，就是添加注解`@EnableAsync`。

```java
package com.lee.redismq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@SpringBootApplication
@EnableAsync
public class RedismqApplication {
    public static void main(String[] args) {
        SpringApplication.run(RedismqApplication.class, args);
    }
}
```



#### 自定义Redis消息队列工具类

我们需要自定义生产者和消费者执行的动作。

```java
package com.lee.mymail2.util;

import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.RedisConnectionFailureException;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.concurrent.TimeUnit;

/**
 * 使用前需要确保能正确连接Redis
 * @author 晓龙coding
 */
@Component
@Slf4j
public class RedisMessageQueueUtil {
    private final String MESSAGE_QUEUE = "message_queue";

    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    /**
     * 作用：将任务加入队列
     * @param obj 实际使用的过程中需要自定义参数
     * @return 是否将任务加入队列
     */
    public boolean push(String obj) {
        try {
            redisTemplate.opsForList().leftPush(MESSAGE_QUEUE, obj);
        } catch (RedisConnectionFailureException e) {
            log.info("redis连接失败");
            return false;
        }
        log.info("成功加入任务: {}", obj);
        return true;
    }

    /**
     * 实际使用的时候需要设定obj的类型，并在consume中调用具体的消费函数
     */
    @Async
    public void consume() {
        while (true) {
            try {
                // 使用时需要修改为符合实际情况的参数
                String name = (String) redisTemplate.opsForList().rightPop(MESSAGE_QUEUE, 365, TimeUnit.DAYS);
                // 调用函数进行消费
                // ...
                System.out.println("消费任务: " + name);
            } catch (RedisConnectionFailureException e) {
                log.info("redis连接失败");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException ex) {
                    throw new RuntimeException(ex);
                }
            }
        }
    }
}
```



#### 自启动消费函数

可以参考我的另一篇文章《SpringBoot自启动程序》。

自定义类，并调用消费函数。

```java
package com.lee.mymail2.config;

import com.lee.mymail2.util.RedisMessageQueueUtil;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;


/**
 * @author 晓龙coding
 */
@Component
public class MyInitializer implements CommandLineRunner {
    @Resource
    private RedisMessageQueueUtil redisMessageQueueUtil;
    @Override
    public void run(String... args) throws Exception {
        // 调用初始化函数
        redisMessageQueueUtil.consume();
    }
}
```

