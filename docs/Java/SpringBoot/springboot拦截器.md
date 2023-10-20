# 拦截器的应用场景

[toc]

## 简介

拦截器就是在执行程序之前，执行某些操作。

拦截器针对URL拦截，它虽然不能想AOP那样针对类、方法实现更细粒度的切面操作，但是也能用于很多场景。



### 案例一：拦截器校验登录权限

`preHandle`函数表示执行某些程序前需要执行的程序。返回值为true表示放行请求，返回值为false表示拦截之后的请求。

```java
package com.lee.myweb.interceptor;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class LoginInterceptor implements HandlerInterceptor {  // 拦截器类需要 implements HandlerInterceptor
    // 业务请求处理之前执行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("authorization");
        if (validate(token) == false) {  // validate是自定义校验规则
            // 后端向前端返回String或JSON数据
            response.setContentType("application/json;charset=utf-8");
            response.getWriter().print("校验失败");
            return false;
        } else {
            return true;
        }
    }
    
    public boolean validate(String token) {
        // 实现校验逻辑...
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
    @Resourse
    private LoginInterceptor interceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(interceptor).addPathPatterns("/lhl/**");
    }
}
```



## 案例二：防止重复提交

针对指定注解标注的方法实现拦截。

下述案例中，拦截器的功能偏向与”限流器“，根据时间判断是否重复提交。在真实业务中，我们可以将用户session+URL缓存在全局对象中（消息队列、ThreadLocal...），当方法执行完毕后

> 注解类

```java
package com.lhl.interceptordemo.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PreventDuplicateSubmit {
    int expirationTimeMillSeconds() default 1000;
}

```

> 拦截器类

```java
package com.lhl.interceptordemo.interceptor;

import com.lhl.interceptordemo.annotation.PreventDuplicateSubmit;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.servlet.http.HttpSession;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

@Component
public class DuplicateSubmitInterceptor implements HandlerInterceptor {
    private final Map<String, Boolean> requestMap = new ConcurrentHashMap<>();
    private final ScheduledExecutorService executorService = Executors.newScheduledThreadPool(1);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (handler instanceof HandlerMethod handlerMethod) {
            PreventDuplicateSubmit annotation = handlerMethod.getMethod().getAnnotation(PreventDuplicateSubmit.class);
            if (annotation != null) {
                HttpSession session = request.getSession();
                String sessionId = session.getId();
                String requestKey = getSessionKey(sessionId, request.getRequestURI());
                if (requestMap.containsKey(requestKey)) {
                    // 重复提交
                    response.setContentType("application/json;charset=utf-8");
                    response.getWriter().write("请勿重复提交");
                    return false;
                }
                // 首次提交，将请求标识存入Map
                requestMap.put(requestKey, true);
                // 延迟一定时间后异步删除请求标识
                scheduleRequestCleanup(requestKey, annotation.expirationTimeMillSeconds());
                return true;
            }
        }
        return true;
    }

    private void scheduleRequestCleanup(String requestKey, int expirationTimeMillSeconds) {
        executorService.schedule(() -> requestMap.remove(requestKey), expirationTimeMillSeconds, TimeUnit.MILLISECONDS);
    }

    private String getSessionKey(String sessionId, String requestURI) {
        return sessionId + "_" + requestURI;
    }
}

```

> config配置类

```java
package com.lhl.interceptordemo.configuration;

import com.lhl.interceptordemo.interceptor.DuplicateSubmitInterceptor;
import jakarta.annotation.Resource;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMVCConfiguration implements WebMvcConfigurer {
    @Resource
    private DuplicateSubmitInterceptor duplicateSubmitInterceptor;
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(duplicateSubmitInterceptor);
    }
}

```

> 业务类

```java
package com.lhl.interceptordemo.controller;

import com.lhl.interceptordemo.annotation.PreventDuplicateSubmit;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping
public class IndexController {
    @PreventDuplicateSubmit
    @RequestMapping
    public String index() {
        return "hello, world!";
    }
}

```

