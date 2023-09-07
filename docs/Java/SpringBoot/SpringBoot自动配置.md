# SpringBoot自动配置（TODO）

[toc]



## 自动配置

SpringBoot的诞生就是为了解决使用过程中繁琐的xml配置。

SpringBoot提供固定的配置，来简化程序员配置依赖。



## 自动装配

SpringBoot中，引入第三方组件的starter包后，自动将第三方组件中的bean对象加载到IOC容器中。



## 流程

1. SpringBoot启动主类上标注`@SpringBootApplication`注解，该注解主要由`@SpringBootConfiguration`, `@EnableAutoConfiguration`和`@ComponentScan`组成。

2. `@SpringBootConfiguration`实际上是`@Configuration`，作用是将启动主类托管给IOC容器。

3. `@ComponentScan`：默认情况下扫描启动类所在包(比如`com.lhl.demo`)及其子包。

4. `@EnableAutoConfiguration`：包含`@Import(AutoConfigurationImportSelector.class)`和`@Import({AutoConfigurationPackages.Registrar.class})`，自动导入配置类，这是SpringBoot自动配置的核心。

5. `AutoConfigurationImportSelector.class`类会扫描所有`classpath`下的`META/spring.factories`。

   补充：`classpath`通常包含的路径有：环境变量、jar文件、JDK中的标准库等。



SpringBoot通过`@EnableAutoConfiguration`开启自动配置。SpringBoot启动时，会去扫描`autoconfigure`包中的`spring.factories`，进而有需要的加载配置类。`spring.factories`存放了常用中间件的配置类。

通常我们只需要去`application.properties`文件中写上对应的配置信息就能使用对应的组件了。

`spring.factories`中的部分类可以加载。假设我们没有引入`redis-starter`的包，那Redis的配置类就不会被加载。具体Spring在实现的时候就是使用**@ConditionalXXX**进行判断的。比如Redis的配置类就会有`@ConditionalOnClass({RedisOperations.class})`的配置，说明当前环境下如果有RedisOperations.class这个字节码，才会去加载Redis的配置类。



## 拓展

如果你写了个工具类，并

spring.factories用于加载项目包外的bean。
