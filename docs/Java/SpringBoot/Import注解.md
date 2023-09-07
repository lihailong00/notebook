# Import注解

Spring中，`@Import`注解的作用是将其他类托管给IOC容器。举个例子：

假定类A能被Spring扫描到并放入IOC容器。

```java
@Component
public class A {}
```

类B上面没有任何注解，按理说类B无法被托管给IOC容器。

```java
public class B {}
```

但是，我们可以让**被托管的类A**主动将类B引入IOC容器。

```java
@Component
@Import({B.class})
public class A {}

public class B {}
```

那么此时类B就被托管给IOC容器啦～

特别注意：**只有被托管的类才有资格主动将别的类引入IOC容器**。上述案例中，如果类A必须有`@Component`注解，才能将类B引入IOC容器中。

