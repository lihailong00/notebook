# Spring接入缓存



## 前言

缓存有多种，redis，caffein......

缓存可以加快查询效率。

在Spring Boot 3以后，可以通过注解的方式引入缓存。

## 环境

1. JDK 21
2. Spring Boot 3.2.0

## 基础知识

1. 缓存对象应该交给IOC容器托管。所谓缓存对象，就是用来存放缓存数据的全局对象。我们可以将一个缓存对象看作是一个巨大的K-V数组。
2. 缓存常用注解有：`@CacheConfig`、`@CachePut`、`@Cacheable`、`@CacheEvict`。建议结合实战代码去理解。

## 依赖

```XML
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!--引入缓存接口-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    
    <!--引入缓存具体框架-->
    <dependency>
        <groupId>com.github.ben-manes.caffeine</groupId>
        <artifactId>caffeine</artifactId>
        <version>3.1.8</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.30</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

## 实战

### 配置类

```Java
package com.lhl.cachedemo.config;

import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.concurrent.TimeUnit;

@Configuration
public class CacheConfig {
    @Bean("caffeineCacheManager")
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                // 设置最后一次写入或访问后经过固定时间过期
                .expireAfterAccess(60, TimeUnit.SECONDS)
                // 初始的缓存空间大小
                .initialCapacity(100)
                // 缓存的最大条数
                .maximumSize(1000));
        return cacheManager;
    }
}
```

### Model

```Java
package com.lhl.cachedemo.model;

import lombok.Data;
import lombok.ToString;

@Data
@ToString
public class UserInfo {
    private Integer id;
    private String name;
    private String sex;
    private Integer age;
}
```

### Controller

```Java
package com.lhl.cachedemo.controller;

import com.lhl.cachedemo.model.UserInfo;
import com.lhl.cachedemo.service.UserInfoService;
import jakarta.annotation.Resource;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping
public class UserInfoController {

    @Resource
    private UserInfoService userInfoService;

    @GetMapping("/userInfo/{id}")
    public Object getUserInfo(@PathVariable Integer id) {
        UserInfo userInfo = userInfoService.getByName(id);
        if (userInfo == null) {
            return "没有该用户";
        }
        return userInfo;
    }

    @PostMapping("/userInfo")
    public Object createUserInfo(@RequestBody UserInfo userInfo) {
        userInfoService.addUserInfo(userInfo);
        return "SUCCESS";
    }

    @PutMapping("/userInfo")
    public Object updateUserInfo(@RequestBody UserInfo userInfo) {
        UserInfo newUserInfo = userInfoService.updateUserInfo(userInfo);
        if (newUserInfo == null){
            return "不存在该用户";
        }
        return newUserInfo;
    }

    @DeleteMapping("/userInfo/{id}")
    public Object deleteUserInfo(@PathVariable Integer id) {
        userInfoService.deleteById(id);
        return "SUCCESS";
    }
}
```

### Service

```Java
package com.lhl.cachedemo.service;

import com.lhl.cachedemo.model.UserInfo;

public interface UserInfoService {

    /**
     * 增加用户信息
     *
     * @param userInfo 用户信息
     */
    void addUserInfo(UserInfo userInfo);

    /**
     * 获取用户信息
     *
     * @param id 用户ID
     * @return 用户信息
     */
    UserInfo getByName(Integer id);

    /**
     * 修改用户信息
     *
     * @param userInfo 用户信息
     * @return 用户信息
     */
    UserInfo updateUserInfo(UserInfo userInfo);

    /**
     * 删除用户信息
     *
     * @param id 用户ID
     */
    void deleteById(Integer id);

}
```

### serviceImpl（最核心！）

将缓存（也可以叫缓存对象）当作一个key-value数组。

函数的参数列表作为key（通常我们会自己指定key），函数的返回值作为value。

`@CachePut`通常用于增/改操作，被修饰的函数无论如何都会执行。

`@Cacheable`通常用于查询操作，当缓存中存在对应的key，则不执行被修饰的函数。

`@CacheEvict`通常用于删除操作。

```Java
package com.lhl.cachedemo.service.impl;

import com.lhl.cachedemo.model.UserInfo;
import com.lhl.cachedemo.service.UserInfoService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
import java.util.HashMap;

@Slf4j
@Service
@CacheConfig(cacheNames = "caffeineCacheManager")  // 指定cache对象的名称
public class UserInfoServiceImpl implements UserInfoService {
    /**
     * 模拟数据库存储数据
     */
    private HashMap<Integer, UserInfo> userInfoMap = new HashMap<>();

    @Override
    @CachePut(key = "#userInfo.id")  // 默认情况下，key是参数列表的参数，value是返回值。我们可以手动指定key和value。
    public UserInfo addUserInfo(UserInfo userInfo) {
        log.info("create");
        userInfoMap.put(userInfo.getId(), userInfo);
        return userInfo;
    }

    @Override
    @Cacheable(key = "#id")
    public UserInfo getByName(Integer id) {
        log.info("get");
        return userInfoMap.get(id);
    }

    @Override
    @CachePut(key = "#userInfo.id")
    public UserInfo updateUserInfo(UserInfo userInfo) {
        log.info("update");
        if (!userInfoMap.containsKey(userInfo.getId())) {
            return null;
        }
        // 取旧的值
        UserInfo oldUserInfo = userInfoMap.get(userInfo.getId());
        // 替换内容
        if (!StringUtils.isEmpty(oldUserInfo.getAge())) {
            oldUserInfo.setAge(userInfo.getAge());
        }
        if (!StringUtils.isEmpty(oldUserInfo.getName())) {
            oldUserInfo.setName(userInfo.getName());
        }
        if (!StringUtils.isEmpty(oldUserInfo.getSex())) {
            oldUserInfo.setSex(userInfo.getSex());
        }
        // 将新的对象存储，更新旧对象信息
        userInfoMap.put(oldUserInfo.getId(), oldUserInfo);
        // 返回新对象信息
        return oldUserInfo;
    }

    @Override
    @CacheEvict(key = "#id")
    public void deleteById(Integer id) {
        log.info("delete");
        userInfoMap.remove(id);
    }
}
```