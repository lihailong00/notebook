# SprintBoot AOP

[toc]

## 参考资料

[参考文档1](https://www.cnblogs.com/sandea/p/11175834.html)

一些重要的概念：

> 切点：应用程序中选择哪些**连接点**进行拦截和处理的匹配规则，所以切点一定是连接点，反之不成立。

> 通知：包含了需要用于多个应用对象的横切行为，也即定义了“什么时候”和“做什么”。

> 连接点：应用程序中潜在的拦截点。

> 切面：切点和通知的组合体，定义了在哪些连接点上执行哪些操作。



在实际开发中，我对于上述概念有简单但不完全正确的理解。

切点：就是很多函数（也可以是类）组成的集合。通常使用`@PointCut()`注解指定函数集合。

通知：一般在函数上添加`@Before`/`@Around`/`@After`等注解，此时注解和函数共同组成了**通知**，从而能够在切点执行的前/后执行我们的函数。



## 关于@Pointcut

`@Pointcut`注解中通过填写参数可以指定切点。参数的类型有以下几种：

1. execution：使用execution表达式可以匹配方法执行的方法签名和参数。

   写法：

   	1) `@Pointcut("execution(* com.example.service.*.*(..))")`。【注】它只会拦截到指定的目录，不会递归拦截子目录中的函数。如果拦截路径为`com.example`，可能一个函数都没法拦截。
   	2) `@Pointcut("execution(* com.example..*")`

   

   解释：

   ```
   execution: 匹配对象是函数
   *: 待匹配函数的对象返回值类型是任意类型
   com.example.service: 匹配指定路径中的函数
   *: 指定所有类
   *: 指定类中的所有函数
   (..): 参数列表任意
   ```

2.  @annotation：匹配所有使用该注解的函数和类。

   写法：`@Pointcut("@annotation(com.lee.xnxydev.aop.LogAnnotation)")`。

3. within：匹配某个类中的所有方法。

   写法：`@Pointcut("within(com.example.service.*)")`。



## 如何在项目中使用AOP

该案例通过自定义注解的方式，使得Pointcut获取到所有添加自定义注解的类和函数（主要是函数）。

1. 引入依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
       <version>2.7.8</version>
   </dependency>
   
   <!--lombok-->
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <scope>provided</scope>
   </dependency>
   ```
   
   
   
2. 自定义注解

```java
package com.lee.uploadfile.common.aop;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyLogAnnotation {
}
```



3. 实现切面类

```java
package com.lee.uploadfile.common.aop;

import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.After;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * @author lhl
 */
@Aspect
@Component
@Slf4j
public class LogAspect {
    @Pointcut("@annotation(com.lee.uploadfile.common.aop.MyLogAnnotation)")
    public void LogPointCut() {
        // @Pointcut注解下的函数不用写内容
    }
    
    // 获取切点
    @Before("LogPointCut()")
    public void before() {
        System.out.println("加载预处理文件");
    }
    
    // 最常使用的是环绕通知
    @Around("logPointCut()")
    public void handleFunc(ProceedingJoinPoint joinPoint) throws Throwable {
        // joinPoint是连接点，也就是函数。它只能用于@Around注解中
        long beginTime = System.currentTimeMillis();
        // 如果不执行proceed函数，则连接点不会执行
        joinPoint.proceed();
        long time = System.currentTimeMillis() - beginTime;
        log.info("函数运行时间：{}", time);
        Signature signature = joinPoint.getSignature();
        log.info("连接点方法名：{}", signature.getName());
        log.info("连接点方法修饰符:{}", signature.getModifiers());
        log.info("连接点所在类的完整类名:{}", signature.getDeclaringTypeName());
        log.info("连接点所在类的 Class 对象:{}", signature.getDeclaringType());
        log.info("参数值列表{}", Arrays.toString(joinPoint.getArgs()));
        // 还可以获取通用参数，例如ip地址等信息
        // ......
    }

    @After("LogPointCut()")
    public void after() {
        System.out.println("清空内存");
    }
}
```



4. 之后就可以在需要拦截的函数或类上添加自定义注解。

