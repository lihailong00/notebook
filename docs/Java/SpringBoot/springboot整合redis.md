# springboot整合redis

[toc]

## 创建springboot空项目

使用idea创建springboot空项目即可。我使用是springboot版本是2.77，jdk是1.8，包管理工具是maven。



## 启动redis

参考我的另一篇文章**《Linux运行redis》**。



## 导入Maven依赖

SpringData是Spring中数据操作的模块，包含对各种数据库的集成。其中对Redis的集成就叫做SpringDataRedis。

SpringDataRedis中提供了RedisTemplate工具类，其中封装了对Redis的各种操作。

备注：lettuce在生产环境中会出现连接异常，我目前还没解决。

```xml
<dependencies>
    <!--springboot 启动器-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <!--springboot web 项目启动器-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!--redis，使用jedis-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>

    <!--springboot 测试工具-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--lombok注解-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>

    <!--json工具-->
    <dependency>
        <groupId>com.alibaba.fastjson2</groupId>
        <artifactId>fastjson2</artifactId>
        <version>2.0.22</version>
    </dependency>
</dependencies>
```



## 配置application.properties

```properties
spring.redis.database=0
spring.redis.host=xxx.xxx.xxx.xxx
spring.redis.port=6379
spring.redis.password=xxxxxx
```



## 自定义redis的序列化方式

Java自带的序列化方式可能会让redis的自增操作报错。不自定义序列化方式也可以让redis的部分功能正常运行。**建议自定义redis的序列化方式。**

使用时，**key必须是String类型**。

```java
package com.lee.redismq.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;

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



## 测试案例

```java
package com.lee.springbootredis.pojo;

import lombok.Data;

@Data
public class User {
    private String name;
    private int age;
}
```



```java
@SpringBootTest
class SpringbootRedisApplicationTests {
    // 导入spring-boot-starter-data-redis依赖后就可以使用这两个对象
    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Test
    void test1() {
        redisTemplate.opsForValue().set("key", 1);
        redisTemplate.opsForValue().increment("key", 1);
        Object key = redisTemplate.opsForValue().get("key");
        System.out.println(key);
    }

    @Test
    void test2() {
        User user = new User();
        user.setName("lhl");
        user.setAge(8);
        String jsonUser = JSON.toJSONString(user);
        // 通常习惯在序列化之前将对象转成json字符串。
        stringRedisTemplate.opsForValue().set("user", jsonUser);
        System.out.println(stringRedisTemplate.opsForValue().get("user"));
    }
}
```

