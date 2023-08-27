# SpringBoot自动配置（TODO）

[toc]



## 什么是自动配置

SpringBoot的诞生就是为了解决使用过程中繁琐的xml配置。

SpringBoot提供固定的配置，来简化程序员配置依赖。



## 流程

SpringBoot通过`@EnableAutoConfiguration`开启自动配置。SpringBoot启动时，会去扫描`autoconfigure`包中的`spring.factories`，进而有需要的加载配置类。`spring.factories`存放了常用中间件的配置类。

通常我们只需要去`application.properties`文件中写上对应的配置信息就能使用对应的组件了。

`spring.factories`中的部分类可以加载。假设我们没有引入`redis-starter`的包，那Redis的配置类就不会被加载。具体Spring在实现的时候就是使用**@ConditionalXXX**进行判断的。比如Redis的配置类就会有`@ConditionalOnClass({RedisOperations.class})`的配置，说明当前环境下如果有RedisOperations.class这个字节码，才会去加载Redis的配置类。



## SpringBoot启动类

`@SpringBootApplication`注解用于SpringBoot启动类，它是有三个注解构成：`@SpringBootConfiguration`、`@EnableAutoConfiguration`和`@ComponentScan`。

`@SpringBootConfiguration`：包含`@Configuration`，向容器中注入该类。

`@EnableAutoConfiguration`：开启自动配置，不用深究该注解中包含其他注解。

`@ComponentScan`：默认扫描主类下的所有包，也可以指定扫描路径。
