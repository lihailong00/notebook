# SpringBoot拦截器

[toc]

## 简介

拦截器就是在执行程序之前，执行某些操作。



## 简单的拦截器案例

> LoginHandler

`preHandle`函数表示执行某些程序前需要执行的程序。返回值为true表示放行请求，返回值为false表示拦截之后的请求。

```java
package com.lee.myweb.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("进入拦截器");
        String token = request.getHeader("authorization");
        if (token == null) {
            // 后端向前端返回String或JSON数据
            response.setContentType("application/json;charset=utf-8");
            response.getWriter().print("token为空");
            return false;
        } else {
            return true;
        }
    }
}
```



> WebMVCConfig

```java
package com.lee.myweb.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMVCConfig implements WebMvcConfigurer {
    @Autowired
    private LoginInterceptor interceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(interceptor).addPathPatterns("/lhl/**");
    }
}
```