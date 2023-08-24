# Spring常见拓展点及应用(TODO)

[toc]



## 背景

Spring启动完成后，我们可以获取Spring启动过程中在不同的时间节点的信息，比如bean初始化后的bean的名字。

实现方式：`implements`特定的`interface`并重写方法，方法中就能获取Spring启动过程中的信息。



```java
// Spring启动
// ......
// 通过反射的方式获得一个对象
Object instance;
if (instance instanceof BeanNameAware) {
    ((BeanNameAware) instance).setBeanName(beanName);
}
// ......
```





## 第一类

这类接口我们通常不在意发生在Spring容器启动的哪个阶段，只需要获取需要的上下文即可。



## ApplicationContextAware

作用：获取Spring容器。

```java
@Data
public class GetSpringInfo implements ApplicationContextAware {
    public ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```



BeanFactoryAware

BeanNameAware

InitializingBean

DisposableBean

BeanPostProcessor

BeanFactoryPostProcessor：懒加载bean

InstantiationAwareBeanPostProcessorAdapter

InstantiationAwareBeanPostProcessor





## 第二类

这类接口我们需要重视回掉函数执行的时机。



### BeanPostProcesser

Bean初始化前执行：`postProcessBeforeInitialization`

Bean初始化后执行：`postProcessAfterInitialization`

原理：





​	
