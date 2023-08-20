# Java 注解

[toc]



## 注解是什么

注解就像一个标签，它可以写在类或方法上面。我们可以通过**反射**机制获取带有注解的类和方法的相关信息。

使用注解时，有以下几个角色：

1. 注解接口：接口中可以定义属性。以下是一个简单的注解的定义。

   ```java
   import java.lang.annotation.ElementType;
   import java.lang.annotation.Retention;
   import java.lang.annotation.RetentionPolicy;
   import java.lang.annotation.Target;
   
   
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.RUNTIME)
   public @interface MyAnnotation {
       String value() default "";
       int mode() default 0;
   }
   ```

2. 注解对象：官网上好像没有这种说法，但是我习惯将类和方法上的注解叫做注解对象。



## 注解使用案例

背景：

假定我们有一个MyAnnotation注解，注解中有一个`String Author`属性。

同时我们有一个Student类，Student类中有很多方法，不同的方法由不同的人编写。我们通过在方法上添加注解的方式区分哪些人编写了哪些方法。

问题：

我想获取某一个人（假定是`lhl`）编写了哪些方法。



> MyAnnotation接口

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @author 晓龙coding
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    String author() default "默认用户";
}
```



> Student类

```java
/**
 * @author 晓龙coding
 */
public class Student {
    private String name;
    private int age;

    @MyAnnotation(author = "lhl")
    public String getName() {
        return name;
    }

    @MyAnnotation(author = "lc")
    public void setName(String name) {
        this.name = name;
    }

    @MyAnnotation(author = "lc")
    public int getAge() {
        return age;
    }

    @MyAnnotation(author = "lhl")
    public void setAge(int age) {
        this.age = age;
    }
}
```



> 演示类

```java
import java.lang.reflect.Method;

/**
 * @author 晓龙coding
 */
public class Main {
    public static void main(String[] args) {
        // 获取Student类对象
        Class<Student> studentClass = Student.class;
        // 获取Student类中的所有函数
        Method[] methods = studentClass.getMethods();
        for (Method method : methods) {
            // 如果当前方法上有注解@MyAnnotation
            if (method.isAnnotationPresent(MyAnnotation.class)) {
                // 获取注解对象
                MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
                // 获取注解的author属性
                String author = annotation.author();
                if ("lhl".equals(author)) {
                    System.out.println("<=================");
                    // 打印带有@MyAnnotation注解，且author属性值为lhl的方法信息
                    System.out.println("方法名:" + method.getName());

                    System.out.println("参数的类型分别为: ");
                    Class<?>[] parameterTypes = method.getParameterTypes();
                    for (Class<?> parameterType : parameterTypes) {
                        System.out.println(parameterType.getName());
                    }

                    System.out.println("方法的范围值为:" + method.getReturnType());
                    System.out.println("=================>\n");
                }
            }
        }
    }
}
```



## 内置注解

有三个，了解即可。

```java
package com.lee.uploadfile.test;

/**
 * 内置注解：@Override @Deprecated @SuppressWarnings
 * @author lhl
 */
public class Father extends Object {
    String name;

    @Override
    public String toString() {
        return super.toString();
    }

    @Deprecated
    public static String getOldName() {
        return "淘汰后的方法";
    }

    public static void main(String[] args) {
        System.out.println(getOldName());
    }
}
```



## 元注解

元注解有四种，主要使用@Target和@Retention。

```java
/**
 * 元注解有四种，主要使用@Target和@Retention
 * @Target 定义注解可以放在哪些地方（方法上、类上...）
 * @Retention 自定义的注解几乎使用RUNTIME，表示注解在运行时都有效
 * @Documented 表示是否将注解生成在JavaDoc中
 * @Inherited 该注解可以被子类继承
 * @author lhl
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@interface MyAnnotation {

}

public class Base {
    @MyAnnotation
    public void func() {
        System.out.println("hello world");
    }
}
```



## 自定义注解

主要使用@Target和@Retention。

注解的参数：参数类型 + 参数名 + ()。

```java
/**
 * 自定义注解：
 * 注解的参数：参数类型 + 参数名 + ()
 * @author lhl
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation {
    // 以下均为注解参数，而不是函数
    String name();
    int age();
    String[] subject();
}
```



