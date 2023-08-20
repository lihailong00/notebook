# Unsafe类源码解析

[toc]



## 是什么

`sun.misc.Unsafe` 类是 Java 标准库中的一个类，提供了一些底层、高度危险的操作，可以绕过 Java 语言的限制，直接操作内存和执行一些不安全的操作。



## 有什么作用

通过unsafe对象可以修改**指定对象**的值。例如：

```java
public class Main {
    private int num = 0;

    public static void main(String[] args) throws InterruptedException, NoSuchFieldException, IllegalAccessException {
        // 通过反射获取unsafe对象
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);

        long offsetNum = unsafe.objectFieldOffset(Main.class.getDeclaredField("num"));

        Main obj = new Main();
        obj.num = 10;
        // 通过unsafe获取obj对象中field的值
        System.out.println("num=" + unsafe.getInt(obj, offsetNum));
        // 通过unsafe修改obj对象中field的值
        unsafe.putInt(obj, offsetNum, 20);
        System.out.println("num=" + obj.num);
    }
}
```





## 常用API

使用unsafe对象调用函数时，通常需要向函数的参数列表传入参数`(Object o, long offset)`。我们使用unsafe对象是**为了操作某个对象的属性**。



1. 获取unsafe对象

```java
// 通过反射获取unsafe对象
Field f = Unsafe.class.getDeclaredField("theUnsafe");
f.setAccessible(true);
Unsafe unsafe = (Unsafe) f.get(null);
```



2. 内存操作：

   ```java
   // 常用API：
   public native int getIntVolatile(Object o, long offset);
   public native int getInt(Object o, long offset);
   public native void putInt(Object o, long offset, int x);
   
   // 具体使用：参考上面的案例
   ```

   

3. CAS

   只能对成员变量进行操作，无法对局部变量进行操作。

   ```java
   // 参数列表：被操作的对象，偏移量（用于定位具体field），期待值，修改后的值
   public final native boolean compareAndSwapInt(Object o, long offset, int expected, int x);
   ```
