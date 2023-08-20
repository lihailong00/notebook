# Spring Boot启动流程

[toc]



@SpringBootApplication注解的启动类。它由@EnableAutoConfiguration，@SpringBootConfiguration，@ComponentScan组成。

添加@EnableAutoConfiguration，启动时就会套入自动配置类（AutoConfigurationSelector），这个类会将所有符合条件的@Configuration配置进行加载。

@SpringBootConfiguration等同于@Configuration，作用是将这个类标记为配置类，并加载到容器中。

@ComponentScan自动扫描并加载符合条件的bean。



Spring主类执行run方法后，会经历4个阶段。

> 服务构建



环境准备

容器创建

填充容器
