# Sping Boot 异步事件，线程池，任务队列

[toc]



## 什么是异步事件

假设有这样的场景：用户打开一次文章时，文章的阅读数+1。如果我们将获取文章和修改文章的阅读数的代码写在一起，则会出现这样的问题。

1. 我们要尽快给用户返回文章信息，以改善阅读体验。然而获取文章的同时伴随着修改文章的阅读数，这会消耗一定的时间。
2. 我们在获取文章的同时，可以临时新创建一个线程，用于修改文章的阅读数。这样就解决了消息延迟的问题。
3. 上述情况中，修改文章阅读数的事件称为异步事件。



## 为什么要使用线程池

在处理异步事件的过程中，Java程序会不断创建新的线程用于修改文章阅读数，这非常消耗系统性能。实际上我们只需要创建一个线程池，让线程池中的线程处理这些异步事件即可。这样解决了线程频繁创建和销毁带来的性能损耗。



## 如何创建异步事件

创建异步事件等价于定义异步执行的函数。

首先我们要在Spring Boot的启动类上加上注解：`@EnableAsync`。

然后我们只需要对异步执行的函数上加上注解：`@Async`。

注意：异步函数的返回值只能是void或Future类型。



@Async 失效的原因：

1. 必须在启动类中增加@EnableAsync注解。
2. 异步类没有被springboot管理，在有异步方法的类上添加@Component注解（或其他注解）且保证可以扫描到异步类。
3. 调用异步函数的函数不能和异步函数在同一个类中。
4. 必须通过ioc容器获得的对象才能调用异步方法，自己new的对象不能调用异步方法。



## 创建线程池

1. 启动类中加上注解：`@EnableAsync`。

2. 创建线程池，并给线程池设定一个名字。线程池中包含任务队列的参数。

   ```java
   package com.lee.threadpool.config;
   
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.scheduling.annotation.EnableAsync;
   import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
   
   import java.util.concurrent.Executor;
   
   /**
    * @author 晓龙coding
    */
   @Configuration
   @EnableAsync
   public class ThreadPoolConfig {
       // 指定bean对象的名字。
       @Bean("taskExecutor")
       public Executor asyncServiceExecutor() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           // 设置核心线程数
           executor.setCorePoolSize(2);
           // 设置最大线程数
           executor.setMaxPoolSize(2);
           //配置【任务队列】的大小
           executor.setQueueCapacity(1000);
           // 设置线程活跃时间（秒）
           executor.setKeepAliveSeconds(60);
           // 设置默认线程名称
           executor.setThreadNamePrefix("认证模块");
           // 等待所有任务结束后再关闭线程池
           executor.setWaitForTasksToCompleteOnShutdown(true);
           //执行初始化
           executor.initialize();
           return executor;
       }
   }
   ```

3. 指定异步方法：

   ```java
   package com.lee.threadpool.service;
   
   import org.springframework.scheduling.annotation.Async;
   import org.springframework.stereotype.Component;
   
   import java.time.LocalDateTime;
   
   @Component
   public class ThreadService {
       // 指定线程池“taskExecutor”
       @Async("taskExecutor")
       // 使用Async注解标注的函数返回值必须是 void 或 Future-like 类型
       public void task() {
           try {
               System.out.println(LocalDateTime.now() + ": hello world!");
               // 睡眠5秒 模拟执行任务
               Thread.sleep(5000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
   }
   ```

4. 调用异步方法：

   ```java
   package com.lee.threadpool.controller;
   
   import com.lee.threadpool.service.ThreadService;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   @RequestMapping("/thread")
   public class ThreadController {
       @Autowired
       private ThreadService threadService;
       @GetMapping("")
       String task() {
           // threadService 中的 task 函数将会被放入线程池中执行
           threadService.task();
           return "访问成功";
       }
   }
   ```

   