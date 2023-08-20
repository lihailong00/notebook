# SpringBoot自启动程序

[toc]



## 前言

有时候我们需要指定某些函数在SpringBoot刚启动的时候就要执行。

对于这种情况，我们通常自己定义一个类，将需要执行的函数放进去即可。



## 代码

自定义类：`MyInitializer`

```java
package com.lee.mymail2.config;

import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;


/**
 * @author 晓龙coding
 */
@Component
public class MyInitializer implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        // 执行具体函数
        // ......
    }
}

```



## 建议

如果想在项目刚启动时一直运行某个程序，达到监听任务的功能，那么`@PostConstruct`注解不能放在Controller中的函数上面，而是要放在service上面。
