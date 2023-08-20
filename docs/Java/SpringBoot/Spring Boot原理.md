# Spring Boot原理

[toc]



## 依赖

在Spring Boot的pom文件中有一个标签`parent`，其中指定了通用的依赖管理和插件管理信息。点进`spring-boot-starter-parent -> spring-boot-dependencies`，我们便能看到有很多依赖信息。这就是很多时候我们在pom文件中导入依赖时不用指定版本的原因，因为父项目中已经指定好，并且经过了完整的测试。但是注意，并不是我们依赖的所有项目都会包含在父项目中。

**建议**：引入有些依赖建议不指定版本，因为Spring官方已经在父项目中指定了稳定的版本。



## starter

`spring-boot-starter`是Spring Boot的核心starter，它包含了Spring框架的基础模块，比如Spring核心、Spring上下文、Spring AOP、Spring Test等。

`spring-boot-starter-web`则是专门用于构建Web应用程序的starter，它包含了Spring MVC、Tomcat、Jackson等Web相关的依赖。

由于`spring-boot-starter-web`中包含了`spring-boot-starter`，所以在引入前者的时候不用引入后者。

同理，spring还定义了其他很多starter，例如：`spring-boot-starter-aop`。



## 自动配置机制

**自动**是指我们只需要引入需要的包，不用管相关的配置，`Spring Boot`会自动帮我们注入相关`bean`。我们可以在需要的地方使用`@Autowired`或者`@Resource`等注解来使用它。



## 条件注解





## 配置优先级

[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config)

文档提供了十余种配置，其底层原理是：每种配置都对应一个`Map`，`Spring Boot`按照优先级从高到低进行扫描获取配置。



> 详解

1. Default properties (specified by setting `SpringApplication.setDefaultProperties`).

   ```java
   @SpringBootApplication
   public class MyApplication {
       public static void main(String[] args) {
           SpringApplication application = new SpringApplication(MyApplication.class);
   
           // 设定参数
           Map<String, Object> params = new HashMap<>();
           params.put("server.port", 8081);
   
           // 传入参数
           application.setDefaultProperties(params);
   
           application.run(args);
       }
   }
   ```

   

2. `@PropertySource`annotations on your `@Configuration` classes. Please note that such property sources are not added to the `Environment` until the application context is being refreshed. This is too late to configure certain properties such as `logging.*` and `spring.main.*` which are read before refresh begins.

   

   启动类上添加`@PropertySource`。

   ```java
   @SpringBootApplication
   @PropertySource("classpath:myapplication.properties")
   public class MythymeApplication {
       public static void main(String[] args) {
           SpringApplication.run(MythymeApplication.class, args);
       }
   }
   ```

   `src/main/resources/`创建`myapplication.properties`，并填写以下内容：

   ```properties
   server.port=8082
   ```

   

3. Config data (such as `application.properties` files).

   `src/main/resources/application.properties`默认文件中填写配置信息。

   

4. OS environment variables.

   从操作系统中获取环境变量，`idea`中也可以临时配置环境变量。

   ![QQ截图20230512113118](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/QQ%E6%88%AA%E5%9B%BE20230512113118.png)

   

   ![QQ截图20230512113432](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/QQ%E6%88%AA%E5%9B%BE20230512113432.png)

   

   

5. Command line arguments.

   这种方式相当于在命令行中输入：`java -jar xxx.jar --server.port=8085`。

   ![QQ截图20230512114434](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/QQ%E6%88%AA%E5%9B%BE20230512114434.png)



