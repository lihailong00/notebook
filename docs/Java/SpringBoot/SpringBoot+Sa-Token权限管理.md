# Spring Boot+Sa-Token权限管理



[toc]





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
# 是否输出操作日志，生产环境一般关闭
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

配置文件`application.properties`中还需要配置私钥。

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

如果我们希望在jwt中塞入一些信息，需要在`StpUtil.login()`函数中传入参数，例如：`StpUtil.login(10001, SaLoginConfig                .setExtra("name", "zhangsan").setExtra("age", 18).setExtra("role", "超级管理员"));`



3. 我们还需要接入Redis。首先引入依赖。

```xml
<!-- 使用redis必用的依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- Sa-Token 整合 Redis （使用 jackson 序列化方式） -->
<!-- redis数据序列化有多种方式，我们通常采用jackson序列化，这样的数据易读，但兼容性少差(目前还没遇到兼容性问题) -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-redis-jackson</artifactId>
    <version>1.37.0</version>
</dependency>
```

然后在`application.properties`中需要配置redis。

```properties
# Redis数据库索引（默认为0）
spring.redis.database=1
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
# spring.redis.password=
# 连接超时时间
spring.redis.timeout=10s
# 连接池最大连接数
spring.redis.lettuce.pool.max-active=200
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=-1ms
# 连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=10
# 连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
```

然后直接使用就行。sa-token会自动将token存入redis。

服务器重启后，只要redis中的token还存在，都会保留用户的登陆状态。



4. 通常还会配置一个全局拦截器，如果用户登陆了，才能访问大多数端口。同时也需要放行一些端口供所有人访问，比如登陆的接口。因此我们需要修改之前关于sa-token的配置类。

   如果用户没有通过拦截器，会报500错误。

```java
package com.lee.satokendemo.config;

import cn.dev33.satoken.interceptor.SaInterceptor;
import cn.dev33.satoken.stp.StpUtil;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class SaTokenConfigure implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 注册 Sa-Token 拦截器，校验规则为 StpUtil.checkLogin() 登录校验。
        registry.addInterceptor(new SaInterceptor(handle -> StpUtil.checkLogin()))
                .addPathPatterns("/**")
                .excludePathPatterns("/acc/doLogin")
          			.excludePathPatterns("/acc/isLogin");;
    }
}
```





5. 下面是一个简单的测试案例：

```java
package com.lee.satokendemo.controller;

import cn.dev33.satoken.stp.SaLoginConfig;
import cn.dev33.satoken.stp.StpUtil;
import cn.dev33.satoken.util.SaResult;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping ("acc")
public class LoginController {
    // 测试登录  ---- http://localhost:8081/acc/doLogin?name=zhang&pwd=123456
    @RequestMapping("doLogin")
    public SaResult doLogin(String name, String pwd) {
        // 此处仅作模拟示例，真实项目需要从数据库中查询数据进行比对
        if("zhang".equals(name) && "123456".equals(pwd)) {
            StpUtil.login(10001, SaLoginConfig
                    .setExtra("name", "zhangsan")
                    .setExtra("age", 18)
                    .setExtra("role", "超级管理员"));  // 这里传入的id建议设置为用户特有的字段，比如uid。
            return SaResult.ok("登录成功");
        }
        return SaResult.error("登录失败");
    }

    // 查询登录状态  ---- http://localhost:8081/acc/isLogin
    @RequestMapping("isLogin")
    public SaResult isLogin() {
        return SaResult.ok("是否登录：" + StpUtil.isLogin());
    }

    // 查询 Token 信息  ---- http://localhost:8081/acc/tokenInfo
    @RequestMapping("tokenInfo")
    public SaResult tokenInfo() {
        return SaResult.data(StpUtil.getTokenInfo());
    }

    // 测试注销  ---- http://localhost:8081/acc/logout
    @RequestMapping("logout")
    public SaResult logout() {
        StpUtil.logout();
        return SaResult.ok();
    }

    // 测试封禁  ---- http://localhost:8081/acc/disable?id=10001
    @RequestMapping("disable")
    public SaResult disable(String id) {
        StpUtil.disable(id, 86400);
        return SaResult.ok("封禁成功, id=" + id);
    }
}

```





## 设计理念

1. 相同用户多次调用登陆接口，会在redis中生成多个token吗？会的！如果删掉最近生成的token，会提示该用户还会登陆吗？会的，即便之前生成的token还保留在redis中。

2. 封禁账号、退出登陆、踢人下线有什么区别？为什么要有踢人下线的功能（个人认为是多端登陆）

   封禁账号`StpUtil.disable(id, 86400)`：会在redis中插入一条记录，并保留原先的记录，用户仍然处于登陆状态，只是无法通过`StpUtil.checkDisable(loginId)`函数。思考为什么会在redis中插入一条记录呢？假定封禁账号是删除原先的token，那么用户重新登陆后就会生成新的token，这样封禁就失效了。可以试试，在redis中删掉该记录，封禁账号就解除了。

   退出登陆：





## todo

1. 封禁账号时，如何查看还有多久解封
2. 封禁部分功能如何实现？
