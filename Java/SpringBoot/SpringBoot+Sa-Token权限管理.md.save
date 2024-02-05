# Spring Boot+Sa-Token权限管理



## 关于Sa-Token

Sa-Token是一个轻量级的权限校验框架，我通常在Spring Boot中集成Sa-Token用于保证系统的安全性。



## 个人使用习惯

1. token的格式是JWT。
2. 引入Redis保存JWT，用于提前废弃token。



## 快速开始

### 一、引入依赖

1. 使用sa-token，需要引入最基本的依赖。[参考链接](https://sa-token.cc/doc.html#/start/example)

```xml
<!-- 如果你使用的是 SpringBoot 3.x，只需要将 sa-token-spring-boot-starter 修改为 sa-token-spring-boot3-starter 即可。 -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot-starter</artifactId>
    <version>1.37.0</version>
</dependency>
```

还需要进行最基本的配置

```properties
############## Sa-Token 配置 (文档: https://sa-token.cc) ##############

# token 名称（同时也是 cookie 名称）
sa-token.token-name=X-Jwt-Token
# token 有效期（单位：秒） 默认30天，-1 代表永久有效
sa-token.timeout=2592000
# token 最低活跃频率（单位：秒），如果 token 超过此时间没有访问系统就会被冻结，默认-1 代表不限制，永不冻结
sa-token.active-timeout=-1
# 是否允许同一账号多地同时登录 （为 true 时允许一起登录, 为 false 时新登录挤掉旧登录）
sa-token.is-concurrent=true
# 在多人登录同一账号时，是否共用一个 token （为 true 时所有登录共用一个 token, 为 false 时每次登录新建一个 token）
sa-token.is-share=true
# token 风格（默认可取值：uuid、simple-uuid、random-32、random-64、random-128、tik）
sa-token.token-style=uuid
# 是否输出操作日志
sa-token.is-log=true
```



2. 我们需要JWT格式的token，需要引入依赖。[参考链接](https://sa-token.cc/doc.html#/plugin/jwt-extend)

```xml
<!-- Sa-Token 整合 jwt -->
<!-- 注意: sa-token-jwt 显式依赖 hutool-jwt 5.7.14 版本，保险起见：你的项目中要么不引入 hutool，要么引入版本 >= 5.7.14 的 hutool 版本。 -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-jwt</artifactId>
    <version>1.37.0</version>
</dependency>
```

还需要配置私钥。

```properties
# jwt秘钥（随便敲一串乱码）
sa-token.jwt-secret-key=23903k2qivfpoar89ppyoe45u0fonzfch932r4jm4ipoaufe09o
```

还需要配置配置类。

```java
package com.lee.satokendemo.config;

import cn.dev33.satoken.jwt.StpLogicJwtForSimple;
import cn.dev33.satoken.stp.StpLogic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SaTokenConfigure {
    // Sa-Token 整合 jwt (Simple 简单模式)，一般来说都使用Simple模式。
    @Bean
    public StpLogic getStpLogicJwt() {
        return new StpLogicJwtForSimple();
    }
}
```



3. 我们还需要接入Redis。首先引入依赖。

```xml
```

