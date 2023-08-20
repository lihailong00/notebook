

# SpringBoot登录注册

[toc]

## 前置类

> Result类（非必须）

```java
import lombok.AllArgsConstructor;
import lombok.Data;
@Data
@AllArgsConstructor
public class Result {
    private boolean success;

    private int code;

    private String msg;

    private Object data;

    public static Result success(Object data) {
        return new Result(true, 200, "success", data);
    }

    public static Result fail(int code, String msg) {
        return new Result(false, code, msg, null);
    }
}
```





## 登录功能

### 前置需求

由于要用到JWT和MD5加密，需要引入以下依赖以及**自定义的JWT工具类（见另一个文件）**。

```xml
<!--md5加密-->
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
</dependency>
<!--JWT-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```



### 功能梳理

1. 用户从前端通过post请求传入`username`和`password`将其封装成`Map`。
2. 从controller传入service。
3. service中初步判断登录条件（验证用户名、密码不为空等），然后对密码进行加盐加密操作（加分项）。
4. 通过用户名和密码这两个属性去数据库中查找，将返回的数据（例如：`name`, `email`）存放在Map中。
5. 使用JWT创建token，并将个人信息（Map形式）和加密时间存放在token中，在redis中以key value的形式存放个人信息（key：token，value：个人信息），同时设置redis中的过期时间（加分项）。
5. 最终在service中返回Result结果。（加分项）

备注：

1. Result类是自定义的。
1. JWT代码参考我的相关文档。
1. 个人信息不仅存放在JWT中，也存放在redis中。



### 前端

#### 前端发送请求

`method: post`

请求体：

```json
{
  "username": "lhl",
  "password": 123
}
```



#### 前端接收请求

数据经过自定义Result类封装。

data是服务器创建的token。

```json
{
  "success": true,
  "code": 200,
  "msg": "success",
  "data": "eyJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7ImFjY291bnQiOiJsaGwiLCJlbWFpbCI6ImxobEBhaWJhaW5jaGVuZyJ9LCJleHAiOjE2NjIzNTE3ODQsImlhdCI6MTY2MjM1MTY5OH0.Cu524IfTfYCo42lH875rblPRkRhBt7o-2IymztfG-rc"
}
```



### 代码

#### Controller层

> LoginController

```java
@RestController
@RequestMapping("/login")
public class LoginController {
    @Autowired
    private LoginService loginService;
    @PostMapping("/login")
    public Result login(@RequestBody Map<String, Object> userInfo) {
        return loginService.login(userInfo);
    }
}
```



#### Service 层

> LoginServiceImpl 类实现

```java
@Service
public class LoginServicImpl implements LoginService {
    @Autowired
    private LoginMapper loginMapper;

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public Result login(Map<String, Object> userInfo) {
        String username = userInfo.get("username").toString();
        String password = userInfo.get("password").toString();
        System.out.println("用户名：" + username);
        System.out.println("密码：" + password);
        Map<String, Object> user = loginMapper.findUser(username, password);
        System.out.println(user);
        // 可以加盐加密

        if (user == null) {
            return Result.fail(250, "用户名或密码不正确");
        } else {
            // 生成token，并在其中装载个人信息和token过期时间
            String token = JWTUtils.createToken(user,24 * 60 * 60L);

            // 调试
            System.out.println("生成token：" + token);

            // 另key是token，value是个人信息，存入redis并设置过期时间
            redisTemplate.opsForValue().set("TOKEN_" + token, JSON.toJSONString(user), 1, TimeUnit.DAYS);
            return Result.success(token);
        }
    }

    @Override
    public Result getUserInfo(String token) {
        // 调试
        System.out.println("验证token：" + token);

        Map<String, Object> stringObjectMap = JWTUtils.checkToken(token);
        if (stringObjectMap == null) {
            return Result.fail(250, "JWT验证失败");
        }
        String userJson = (String) redisTemplate.opsForValue().get("TOKEN_" + token);
        if (StringUtils.isBlank(userJson)) {
            return Result.fail(250, "redis中token过期");
        }
        Map<String, Object> user = JSON.parseObject(userJson, Map.class);
        return Result.success(user);
    }
}
```



#### Mapper层

> LoginMapper 接口

```java
import org.apache.ibatis.annotations.Param;
import java.util.Map;

public interface LoginMapper {
    Map<String, Object> findUser(@Param("username") String username, @Param("password") String password);
}
```



> LoginMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lee.mybatis03.mapper.LoginMapper">
    <select id="findUser" resultType="java.util.Map">
        select account, email from blog_sys_user
        where account = #{username} and password = #{password}
    </select>
</mapper>
```

备注，数据库中的`account`和`email`是登录成功后需要返回的个人信息。





## 登录后获取个人信息

### 前置需求

同登录功能。



### 功能梳理

登录成功后，客户端拿到服务器生成的token。先将token发送给服务器，服务器校验成功后返回数据。



### 前端

#### 前端发送数据

接口url：`/login/getinfo`

请求方式：`GET`

请求参数：

| 参数名称      | 参数类型 | 说明              |
| ------------- | -------- | ----------------- |
| Authorization | String   | 头部信息（TOKEN） |

```json
GET http://localhost:8080/login/getinfo
Accept: application/json
Authorization: eyJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7ImFjY291bnQiOiJsaGwiLCJlbWFpbCI6ImxobEBhaWJhaW5jaGVuZyJ9LCJleHAiOjE2NjIzNjQ3MjgsImlhdCI6MTY2MjM2NDY0MX0.SQ7W7RSuM9jiPT_LXqbXaUdyziAMOxNdnYCyuHqennA
```



#### 前端接收数据

```json
{
  "success": true,
  "code": 200,
  "msg": "success",
  "data": {
    "account": "lhl",
    "email": "lhl@aibaincheng"
  }
}
```



### 代码

由于不需要和数据库打交道，因此只需要controller和service。

#### controller

```java
@RestController
@RequestMapping("/login")
public class LoginController {
    @Autowired
    private LoginService loginService;
    @GetMapping("/getinfo")
    public Result getUserInfo(@RequestHeader("Authorization") String token) {
        return loginService.getUserInfo(token);
    }
}
```



#### Service

```java
@Service
public class LoginServicImpl implements LoginService {
    @Autowired
    private LoginMapper loginMapper;

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public Result getUserInfo(String token) {
        // 调试
        System.out.println("验证token：" + token);

        Map<String, Object> stringObjectMap = JWTUtils.checkToken(token);
        if (stringObjectMap == null) {
            return Result.fail(250, "JWT验证失败");
        }
        String userJson = (String) redisTemplate.opsForValue().get("TOKEN_" + token);
        if (StringUtils.isBlank(userJson)) {
            return Result.fail(250, "redis中token过期");
        }
        Map<String, Object> user = JSON.parseObject(userJson, Map.class);
        return Result.success(user);
    }
}
```



## 退出登录

### 功能梳理

前端发送token，服务器接收后在redis中删除该token即可。

### 前端

#### 前端发送数据

接口url：`/login/out`

请求方式：`GET`

请求参数：

| 参数名称      | 参数类型 | 说明              |
| ------------- | -------- | ----------------- |
| Authorization | String   | 头部信息（TOKEN） |

```json
GET http://localhost:8080/login/out
Accept: application/json
Authorization: eyJhbGciOiJIUzI1NiJ9.eyJkYXRhIjp7ImFjY291bnQiOiJsaGwiLCJlbWFpbCI6ImxobEBhaWJhaW5jaGVuZyJ9LCJleHAiOjE2NjIzNjQ3MjgsImlhdCI6MTY2MjM2NDY0MX0.SQ7W7RSuM9jiPT_LXqbXaUdyziAMOxNdnYCyuHqennA
```



#### 前端接收信息

```json

```



### 代码

#### controller

> LoginController

```java
@RestController
@RequestMapping("/login")
public class LoginController {
    @Autowired
    private LoginService loginService;
    @GetMapping("/out")
    public Result logout() {
        return loginService.logout();
    }
}
```



#### service

> LoginServiceImpl

```java
@Service
public class LoginServicImpl implements LoginService {
    @Autowired
    private LoginMapper loginMapper;

    @Autowired
    private RedisTemplate redisTemplate;

    @Override
    public Result logout(String token) {
        redisTemplate.delete("TOKEN_" + token);
        return Result.success("退出成功！");
    }
}
```





## 注册功能

### 前置需求

1. 输入注册信息，先检查输入信息是否合法
2. 在数据库中查询用户名是否存在
3. 将数据写入数据库，注意要使用`@Transactional`注解。
4. 将个人部分信息通过JWT存放在token中。
5. 在redis中存入token。



### controller

> RegisterController

```java
@RestController
@RequestMapping("/register")
public class RegisterController {
    @Autowired
    private RegisterService registerService;
    @PostMapping()
    public Result register(@RequestBody Map<String, Object> user) {
        return registerService.register(user);
    }
}
```



### service

> RegisterServiceImpl

```java
@Service
@Transactional
public class RegisterServiceImpl implements RegisterService {
    @Autowired
    private RegisterMapper registerMapper;
    @Autowired
    private RedisTemplate redisTemplate;
    @Override
    public Result register(Map<String, Object> user) {
        String username = user.get("username").toString();
        String password = user.get("password").toString();
        String email = user.get("email").toString();


        // 验证密码合法性
        if (StringUtils.isBlank(username) || StringUtils.isBlank(password)) {
            return Result.fail(250, "输入信息不合法");
        }

        // 验证密码是否在数据库中存在
        String uName = registerMapper.getUsername(username);
        if (uName != null) {
            return Result.fail(250, "用户名已存在！");
        }


        // 向数据库中插入数据
        registerMapper.insertUser(username, password, email);


        // 使用JWT创建token，并将username和email存放在token中
        Map<String, Object> userInfo = new HashMap<>();
        userInfo.put("username", username);
        userInfo.put("email", email);
        String token = JWTUtils.createToken(userInfo,24 * 60 * 60L);


        // 在redis中存放token
        redisTemplate.opsForValue().set("TOKEN_" + token, JSON.toJSONString(userInfo), 1, TimeUnit.DAYS);
        return Result.success(token);
    }
}
```



### mapper

> RegisterMapper接口

```java
package com.lee.mybatis03.mapper;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

public interface RegisterMapper {
    String getUsername(@Param("username") String username);

    void insertUser(@Param("username") String username,
                    @Param("password") String password,
                    @Param("email") String email);
}
```



> RegisterMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lee.mybatis03.mapper.RegisterMapper">
    <insert id="insertUser">
        insert into blog_sys_user (account, password, email) values (#{username}, #{password}, #{email})
    </insert>
    <select id="getUsername" resultType="java.lang.String">
        select account from blog_sys_user where account = #{username}
    </select>
</mapper>	
```



## 登录拦截器

### 代码

> 拦截类中

```java
@Component
@Slf4j  // lombok 中
public class LoginInterceptor implements HandlerInterceptor {
    /**
     * 在执行controller方法之前执行。
     */
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        /**
         * 1. 判断接口路径是否为HandlerMethod(controller中的方法)
         * 2. 判断token是否为空，如果为空，未登录
         * 3. 如果token不为空，登录验证(JWT checkToken + Redis)
         * 4. 如果成功，放行
         */

        // handler可能是资源
        // springboot 访问静态资源默认去classpath下的static目录
        if (!(handler instanceof HandlerMethod)) {
            return true;
        }
        String token = request.getHeader("Authorization");

        // 打印日志
        log.info("====================request start====================");
        String requestURL = request.getRequestURI();
        log.info("request url:{}", requestURL);
        log.info("request method:{}", request.getMethod());
        log.info("token:{}", token);
        log.info("====================request end  ====================");

        if (!StringUtils.isBlank(token)) {
            // 浏览器识别返回信息是Json
            response.setContentType("application/json;charset=utf-8");

            // 返回响应信息
            Result result = Result.fail(250, "未登录");
            response.getWriter().println(JSON.toJSONString(result));

            return false;
        }

        Map<String, Object> stringObjectMap = JWTUtils.checkToken(token);
        if (stringObjectMap == null) {
            response.setContentType("application/json;charset=utf-8");
            Result result = Result.fail(250, "未登录");
            response.getWriter().println(JSON.toJSONString(result));
            return false;
        }

        return true;
    }
}
```



> 配置类中

```java
@Configuration
public class WebMVCConfig implements WebMvcConfigurer {
    @Autowired
    private LoginInterceptor interceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 假定拦截 /test 目录的controller
        registry.addInterceptor(interceptor).addPathPatterns("/test");

        // 拦截除了test目录的
        // registry.addInterceptor(interceptor).addPathPatterns("**").excludePathPatterns("/test");
    }
}
```



#### controller

> TestController

```java
@RestController
@RequestMapping("/test")
public class TestController {
    @RequestMapping
    public String test() {
        return "test界面";
    }
}
```



#### 测试

访问：`http://localhost:8080/test`





## 思考

为什么要将token存放在redis中？
