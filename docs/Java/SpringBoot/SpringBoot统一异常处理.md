# SpringBoot统一异常处理

[toc]

## 思路

1. 创建一个异常处理类（就是一个普通的类），并在类上添加注解`@RestControllerAdvice`，如果返回值不是JSON，就添加`@ControllerAdvice`。

2. 类中的方法上添加`@ExceptionHandler(某个类对象)`和`@ResponseBody`。

3. 当controller层的方法抛出异常后，会匹配异常处理类中的方法。优先匹配子类。
4. `@RestControllerAdvice`只能处理controller层的异常。



## 案例

1. controller层中的方法抛出异常。

```java
package com.lee.exceptiondemo.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping
public class IndexController {
    @RequestMapping("index")
    public String getInfo() {
        int a = 1 / 0;
        return "hello, world!";
    }
}
```



2. 创建统一异常处理类

```java
package com.lee.exceptiondemo.exception;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public String handleException() {
        return "未知异常";
    }

    @ExceptionHandler(RuntimeException.class)
    public String handleRuntimeException(ArithmeticException exception) {
        return "运行阶段报错";
    }

    @ExceptionHandler(ArithmeticException.class)
    public String handleArithmeticException(ArithmeticException exception) {
        return "算数运算报错";
    }
}
```



3. 由于controller层的方法抛出`ArithmeticException`异常，因此会优先匹配异常处理类中的`ArithmeticException`异常。如果没有匹配成功，则逐次匹配上一级的异常。

   也即匹配优先顺序为：`ArithmeticException` > `RuntimeException` > `Exception`。



## 更进一步

自定义注解类中，方法的返回值不仅可以为String，还可以自定义类型。
