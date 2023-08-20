# Java反射

[toc]



## 是什么

反射是Java语言的一个特性，用于**运行时动态操作类、对象、方法、字段**等信息。

使用反射要谨慎，因为反射会绕过一些安全机制，比如通过**反射可以访问私有成员**。



## 使用案例

> 人类

```java
public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void sayHello() {
        System.out.println("Hello, I'm " + name + ", " + age + " years old.");
    }

    private void sayGood() {
        System.out.println("I'm" + name + ", " + age + " years old. I'm good!!!");
    }

    public static void sayStatic(String str, int num) {
        System.out.println("This is a static method!");
        System.out.println("Your str is " + str);
        System.out.println("Your num is " + num);
    }
}
```



> 主类

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws Exception {
        // 获取类的Class对象
        Class<?> clazz = Class.forName("org.lee._reflection.Person");
        // 另一种获取Class对象的操作
        // Class<?> clazz = Person.class;

        
        // 动态创建对象
        Constructor<?> constructor = clazz.getConstructor(String.class, int.class);
        Object person = constructor.newInstance("Alice", 25);
        // 也可以不通过constructor创建对象，不过这种情况只能调用无参构造函数
        Object obj = clazz.newInstance();

        
        // 获取public方法
        Method sayHelloMethod = clazz.getMethod("sayHello");
        // 获取所有修饰符的方法(private, protected, public...)，并设置为可访问
        Method sayGood = clazz.getDeclaredMethod("sayGood");
        sayGood.setAccessible(true);
        // 同理，也可以获取静态方法。并设置函数参数
        Method sayStatic = clazz.getMethod("sayStatic", String.class, int.class);


        // 调用person对象中的某个方法，函数参数紧跟obj后面
        sayHelloMethod.invoke(person);
        sayGood.invoke(person);
        // 静态方法传参时，obj参数设为null即可。
        sayStatic.invoke(null, "longcoding", 8);


        // 访问字段
        Field nameField = clazz.getDeclaredField("name");
        // 设置字段可访问(即便该字段是private)
        nameField.setAccessible(true);
        String name = (String) nameField.get(person);
        System.out.println("Name: " + name);

        
        // 修改字段的值
        nameField.set(person, "Bob");
        name = (String) nameField.get(person);
        System.out.println("Modified Name: " + name);
    }
}
```



## Java实现反射机制的底层原理

Java程序运行时，会将类相关的信息放入元空间（它不在JVM中，而是在本地内存中）。

具体来说，每个类加载到内存中后，都会在元空间中对应一个实例对象（instanceKlazz）。

假定有一个Person类，则Person.class是一个引用，指向元空间中的Person类的`instanceKlazz`。我们只能通过Person.class访问该对象。

通过Person.class，可以调用很多方法：`getName()`,`getDeclaredFields()`,`getMethods()`。通过这些方法，就可以访问该类对应的`instanceKlazz`对象中的具体数据。

