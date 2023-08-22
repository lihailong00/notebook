# Spring重要类

[toc]



## ApplicationContextAware

spring中，普通bean的注入可以使用`@Resource`，容器bean的注入可以使用`ApplicationContextAware`。实现方式是让当前类`implements`接口`ApplicationContextAware`，并重写方法`setApplicationContext`。



