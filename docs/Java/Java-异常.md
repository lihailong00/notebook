# Java-异常

[toc]



## 异常是什么

`Throwable`是所有**错误**和**异常**的超类。只有当对象是`Throwable`或它的子类时，JVM或`throw`语句才能抛出异常。

`catch`语句中的参数必须是`Throwable`或其子类。



## 异常的分类

1. `Error`：严重的错误，不能手动解决。
2. `Exception`：可处理。
   1. 运行时异常`RuntimeException`，可处理，也可不处理。
      - 空指针异常
      - 数组越界异常
      - 类型转换异常
      - 数字格式化异常
      - 算数异常
   2. 检查时异常`CheckedException`：检查时异常，必须处理。
      - `IOException`



> 不用处理异常

```java
public static void main(String[] args) {
    throw new RuntimeException();
}
```





> 必须处理异常

```java
public static void main(String[] args) throws IOException {
    throw new IOException();
}
```





## 异常的产生和传递

程序员可手动抛出异常。

传递：按照方法的调用链反向传递。如果始终没有处理，则交由JVM进行默认异常处理（打印跟踪堆栈信息，并停止运行）。



## 处理异常

`try`：执行可能产生异常的代码。

`catch`：捕获对应`try`中的异常。

`finally`：无论是否发生异常，都会执行。除非直接退出JVM。（例如`System.exit(0);`）

`throw`：程序员手动抛出异常。

`throws`：声明方法可能抛出的异常。



`try-catch-catch`：匹配第一个catch，且遵循子前父后。

`try-finally`不能捕获异常，会把异常向上抛。



## 自定义异常



```java
package com.lee;

class AgeException extends RuntimeException {
    public AgeException(String message) {
        super(message);
    }
}

class Person {
    private int age;
    public void setAge(int age) {
        if (age > 150) {
            throw new AgeException("非法年龄！");
        }
        this.age = age;
    }

    public int getAge() {
        return this.age;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person();
        person.setAge(1000);
        System.out.println(person.getAge());
    }
}
```



## 其他

子类不能抛出更宽的异常。
