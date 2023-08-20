# RateLimiter限流

[toc]



## 引入依赖

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```



## 代码

```java
package com.lee.mylimiter.controller;

import com.google.common.util.concurrent.RateLimiter;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author 晓龙coding
 */
@RestController
@RequestMapping("")
public class IndexController {
    /**
     * rateLimiter对象一定要在函数外面
     * 令牌桶每秒生成0.5个令牌，也即每2秒生成一个令牌
     */
    private static RateLimiter rateLimiter = RateLimiter.create(0.5);
    private AtomicInteger count = new AtomicInteger(0);
    @RequestMapping("/acquire")
    public String acquire() {
        // 获取1个令牌，如果没有令牌，则阻塞
        rateLimiter.acquire();
        count.incrementAndGet();
        return "count=" + count.get();
    }

    @RequestMapping("tryacquire")
    public String tryAcquire() {
        // 尝试获取1个令牌。如果获取失败，立即返回结果。
        if (rateLimiter.tryAcquire()) {
            count.incrementAndGet();
            return "获取成功，count=" + count.get();
        }
        return "获取失败，count=" + count.get();
    }
}
```



## APO+注解优化版代码

功能：提供了3种基于令牌桶的单机限流方案。

1. 对某个方法限流。
2. 对某个ip限流。
3. 对方法+ip限流。



> 限流工具类

```java
package com.lee.jwccourse.util;

import com.google.common.util.concurrent.RateLimiter;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author 晓龙coding
 */
@Component
public class MyRateLimiterUtil {
    private static final Map<String, RateLimiter> rateLimiters = new ConcurrentHashMap<>();

    /**
     *
     * @param ip 请求的ip地址
     * @param method 注解修饰的方法名
     * @param permits 单位时间通过的次数
     * @param duration 单位时间的长度，单位：秒
     * @return 是否成功通过限流算法
     */
    public static boolean tryAcquire(String ip, Method method, int permits, int duration) {
        // 获取函数的唯一表示作为key
        String className = method.getClass().getName();
        String methodName = method.getName();
        int modifiers = method.getModifiers();
        String key = ip + className + methodName + modifiers;
        RateLimiter rateLimiter = rateLimiters.get(key);
        if (rateLimiter == null) {
            rateLimiter = RateLimiter.create(1.0 * permits / duration);
            rateLimiters.put(key, rateLimiter);
        }
        return rateLimiter.tryAcquire();
    }

    /**
     * 对方法进行限流
     * @param method 注解修饰的方法名
     * @param permits 单位时间通过的次数
     * @param duration 单位时间的长度，单位：秒
     * @return 是否成功通过限流算法
     */
    public static boolean tryAcquire(Method method, int permits, int duration) {
        // 获取函数的唯一表示作为key
        String className = method.getClass().getName();
        String methodName = method.getName();
        int modifiers = method.getModifiers();
        String key = className + methodName + modifiers;

        RateLimiter rateLimiter = rateLimiters.get(key);
        if (rateLimiter == null) {
            rateLimiter = RateLimiter.create(1.0 * permits / duration);
            rateLimiters.put(key, rateLimiter);
        }
        return rateLimiter.tryAcquire();
    }

    /**
     * 对ip全局限流
     * @param ip 请求ip来源
     * @param permits
     * @param duration
     * @return
     */
    public static boolean tryAcquire(String ip, int permits, int duration) {
        RateLimiter rateLimiter = rateLimiters.get(ip);
        if (rateLimiter == null) {
            rateLimiter = RateLimiter.create(1.0 * permits / duration);
            rateLimiters.put(ip, rateLimiter);
        }
        return rateLimiter.tryAcquire();
    }
}
```



> 获取IP工具类

```java
package com.lee.jwccourse.util;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.ObjectUtils;

import javax.servlet.http.HttpServletRequest;
import java.net.InetAddress;
import java.net.UnknownHostException;
 
 
/**
 * @author 晓龙coding
 */
public class IPUtil {
    private static final Logger logger = LoggerFactory.getLogger(IPUtil.class);
    private static final String IP_UTILS_FLAG = ",";
    private static final String UNKNOWN = "unknown";
    private static final String LOCALHOST_IP = "0:0:0:0:0:0:0:1";
    private static final String LOCALHOST_IP1 = "127.0.0.1";
 
    /**
     * 获取IP地址
     * <p>
     * 使用Nginx等反向代理软件， 则不能通过request.getRemoteAddr()获取IP地址
     * 如果使用了多级反向代理的话，X-Forwarded-For的值并不止一个，而是一串IP地址，X-Forwarded-For中第一个非unknown的有效IP字符串，则为真实IP地址
     */
    public static String getIpAddr(HttpServletRequest request) {
        String ip = null;
        try {
            //以下两个获取在k8s中，将真实的客户端IP，放到了x-Original-Forwarded-For。而将WAF的回源地址放到了 x-Forwarded-For了。
            ip = request.getHeader("X-Original-Forwarded-For");
            if (ObjectUtils.isEmpty(ip) || UNKNOWN.equalsIgnoreCase(ip)) {
                ip = request.getHeader("X-Forwarded-For");
            }
            //获取nginx等代理的ip
            if (ObjectUtils.isEmpty(ip) || UNKNOWN.equalsIgnoreCase(ip)) {
                ip = request.getHeader("x-forwarded-for");
            }
            if (ObjectUtils.isEmpty(ip) || UNKNOWN.equalsIgnoreCase(ip)) {
                ip = request.getHeader("Proxy-Client-IP");
            }
            if (ObjectUtils.isEmpty(ip) || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
                ip = request.getHeader("WL-Proxy-Client-IP");
            }
            if (ObjectUtils.isEmpty(ip) || UNKNOWN.equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_CLIENT_IP");
            }
            if (ObjectUtils.isEmpty(ip) || UNKNOWN.equalsIgnoreCase(ip)) {
                ip = request.getHeader("HTTP_X_FORWARDED_FOR");
            }
            //兼容k8s集群获取ip
            if (ObjectUtils.isEmpty(ip) || UNKNOWN.equalsIgnoreCase(ip)) {
                ip = request.getRemoteAddr();
                if (LOCALHOST_IP1.equalsIgnoreCase(ip) || LOCALHOST_IP.equalsIgnoreCase(ip)) {
                    //根据网卡取本机配置的IP
                    InetAddress iNet = null;
                    try {
                        iNet = InetAddress.getLocalHost();
                    } catch (UnknownHostException e) {
                        logger.error("getClientIp error: {}", String.valueOf(e));
                    }
                    assert iNet != null;
                    ip = iNet.getHostAddress();
                }
            }
        } catch (Exception e) {
            logger.error("IPUtils ERROR ", e);
        }
        //使用代理，则获取第一个IP地址
        if (!ObjectUtils.isEmpty(ip) && ip.indexOf(IP_UTILS_FLAG) > 0) {
            ip = ip.substring(0, ip.indexOf(IP_UTILS_FLAG));
        }
        return ip;
    }
}
```





> 注解类

```java
package com.lee.jwccourse.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @author 晓龙coding
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyRateLimiterAnnotation {

    enum Type {
        /**
         * 对单个接口根据IP限流
         */
        IP_API,
        /**
         * 仅对单个接口限流
         */
        API,
        /**
         * 对IP进行全局限流
         */
        IP
    }

    Type type() default Type.IP_API;

    /**
     * 单位时间运行访问量
     * @return
     */
    int permits() default 1;

    /**
     * 单位时间有多少秒
     * @return
     */
    int duration() default 1;
}
```



> 切面类

```java
package com.lee.jwccourse.aop;

import com.lee.jwccourse.annotation.MyRateLimiterAnnotation;
import com.lee.jwccourse.util.IPUtil;
import com.lee.jwccourse.util.MyRateLimiterUtil;
import com.lee.jwccourse.vo.ResponseResult;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;

/**
 * @author 晓龙coding
 */
@Aspect
@Component
@Slf4j
public class MyRateLimiterAspect {
    @Pointcut("@annotation(com.lee.jwccourse.annotation.MyRateLimiterAnnotation)")
    public void rateLimiterPointCut() {
    }

    @Resource
    private HttpServletRequest request;
    @Around("rateLimiterPointCut()")
    public Object handleFunc(ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        MyRateLimiterAnnotation annotation = method.getAnnotation(MyRateLimiterAnnotation.class);

        MyRateLimiterAnnotation.Type type = annotation.type();
        int permits = annotation.permits();
        int duration = annotation.duration();

        if (type == MyRateLimiterAnnotation.Type.IP_API) {
            String ipAddress = IPUtil.getIpAddr(request);
            log.info("函数名:{}", method.getName());
            log.info("ip:{}", ipAddress);
            boolean success = MyRateLimiterUtil.tryAcquire(ipAddress, method, permits, duration);
            if (!success) {
                // 返回失败信息
            }
        }
        else if (type == MyRateLimiterAnnotation.Type.IP) {
            String ipAddress = IPUtil.getIpAddr(request);
            boolean success = MyRateLimiterUtil.tryAcquire(ipAddress, permits, duration);
            if (!success) {
                // 返回失败信息
            }
        }
        else if (type == MyRateLimiterAnnotation.Type.API) {
            boolean success = MyRateLimiterUtil.tryAcquire(method, permits, duration);
            if (!success) {
                // 返回失败信息
            }
        }
        return joinPoint.proceed();
    }
}
```



> 使用案例

```java
// 对方法+ip限流
@MyRateLimiterAnnotation(type = MyRateLimiterAnnotation.Type.IP_API, permits = 1, duration = 5)
@PostMapping("/cancel")
public ResponseResult cancelCourse(@RequestBody Map<String, String> params) {
    // ...... 省略
}

// 对ip限流
@MyRateLimiterAnnotation(type = MyRateLimiterAnnotation.Type.IP, permits = 1, duration = 5)
public void func2() {}

// 对方法限流
@MyRateLimiterAnnotation(type = MyRateLimiterAnnotation.Type.API, permits = 1, duration = 5)
public void func3() {}
```

