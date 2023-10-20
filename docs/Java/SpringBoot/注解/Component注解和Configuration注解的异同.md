# Component注解和Configuration注解的异同

[toc]



## 速记

@Component是lite模式，类直接托管给IOC容器。不同@Bean修饰的方法相互引用时，无法做到单例。

@Configuration是full模式，生成代理类托管给IOC容器。不同@Bean修饰的方法相互引用时，可以做到单例。



## 配置类的两种模式

配置类分为lite模式和full模式。



### lite模式

> lite模式包含以下类：

- 没有被@Configuration修饰，被@Component修饰
- 没有被@Configuration修饰，被@ComponentScan修饰
- 没有被@Configuration修饰，被@Import修饰
- 没有被@Configuration修饰，被@ImportResource修饰
- 没有任何Spring相关注解，类里面有@Bean修饰的方法
- 被@Configuration修饰，但是属性proxyBeanMethods = false



> lite模式的特征：

- lite模式下的配置类不生成代理，原始类型进入容器
- lite模式下的@Bean方法可以是private和final
- 单例scope下不同@Bean方法引用时无法做到单例



### full模式

> full模式包含以下类：

- 被@Configuration修饰，且属性proxyBeanMethods = true(proxyBeanMethods 默认为true)



> full模式的特征：

- full模式下的配置类会被CGLIB代理生成代理类取代原始类型(在容器中)
- full模式下的@Bean方法不能是private和final
- 单例scope下不同@Bean方法可以互相引用，达到单实例的语义