# Swagger教程

[toc]



## 环境

- JDK 8
- Spring Boot 2.7.16



## 依赖

```xml
<!--版本选2.9.2-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!--版本选2.9.2-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```



## 配置类

```java
package com.example.swaggerdemo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
@Profile({"test", "dev"})
public class SpringFoxConfig {                                    
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
          .select()                                  
          .apis(RequestHandlerSelectors.any())
          .paths(PathSelectors.any())
          .build();                                           
    }
}
```



```java
package com.example.swaggerdemo.config;

import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())  // 自定义拦截器
                .addPathPatterns("/**")
                .excludePathPatterns("/swagger-resources/**", "/webjars/**", "/v2/**", "/swagger-ui.html/**");  // 如果有拦截器，需要放开这几个路径
    }
    
    
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```



## 配置文件

```yml
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
  profiles:
    active: dev
```



## 案例

Swagger UI地址：http://localhost:8080/swagger-ui.html

核心注解：

```java
@Api(value = "用户接口")  // 用于controller类上
@ApiOperation("获取用户")  // 用于controller中的方法上
@ApiImplicitParams({
        @ApiImplicitParam(name = "token", value = "用户token"),
        @ApiImplicitParam(name = "userId", value = "用户ID", required = true)
})  // 用于controller中的方法上，解释参数列表中的每个参数
@ApiModel("获取用户的请求")  // 用于模型类上
@ApiModelProperty("用户id")  // 用于模型类的属性上
```



### controller

```java
package com.example.swaggerdemo.controller;

import com.example.swaggerdemo.request.UpdateUserNameRequest;
import com.example.swaggerdemo.response.GetUserResponse;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("user")
@Api(value = "用户接口")
public class UserController {
    @GetMapping("getuser")
    @ApiOperation("获取用户")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "token", value = "用户token"),
            @ApiImplicitParam(name = "userId", value = "用户ID", required = true)
    })
    public GetUserResponse getUser(@RequestHeader("token") String token, @RequestParam("userId") Long userId) {
        return new GetUserResponse(userId, "访问成功");
    }

    @GetMapping("updateusername")
    @ApiOperation("更新用户")
    // 不需要@ApiImplicitParams
    public String updateUsername(@RequestBody UpdateUserNameRequest updateUserNameRequest) {
        return "update success!";
    }
}
```



### model

```java
package com.example.swaggerdemo.request;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

@Data
@ApiModel("获取用户的请求")
public class UpdateUserNameRequest {
    @ApiModelProperty("用户id")
    private Long userId;
    @ApiModelProperty("用户名")
    private String username;
}
```



```java
package com.example.swaggerdemo.response;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
@ApiModel("获取用户的请求")
public class GetUserResponse {
    @ApiModelProperty("用户id")
    private Long userId;
    @ApiModelProperty("响应信息")
    private String message;
}
```

