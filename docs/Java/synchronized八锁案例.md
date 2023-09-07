# synchronized八锁案例

[toc]



## 案例

假定有如下代码：

```java
class Phone {
    public synchronized void printA() {
        try { Thread.sleep(2000); } catch (Exception e) { throw new RuntimeException(e); }
        System.out.println("AAA");
    }
    public synchronized void printB() {
        System.out.println("BBB");
    }
    public void printC() {
        System.out.println("CCC");
    }
}

public class Main {
    public static void main(String[] args) {
        // 创建phone对象
        // ...
        // 启动1线程
        // ...
        // 暂停500ms
        try { Thread.sleep(500); } catch (Exception e) { throw new RuntimeException(e); }
        // 启动2线程
        // ...
    }
}
```



八锁问题：

默认情况下只创建一个手机对象。

1. 假定1线程中没有Sleep函数。1线程调用printA，2线程调用printB。请问打印结果？

```
AAA
BBB
```



2. 1线程调用printA，2线程调用printB。请问打印结果？

```
AAA
BBB
```

解释：synchronized方法实际上锁的是this对象。线程1和线程2使用的锁都来自手机对象。因此一定是先AAA后BBB。



3. 1线程调用printA，2线程调用printC。请问打印结果？

```
CCC
AAA
```

解释：线程2嗲用printC并不会和线程1竞争。因此先执行CCC，后执行AAA。



4. 两部手机，1线程调用手机1的printA，2线程调用手机2的printB。请问打印结果？

```
BBB
AAA
```

虽然线程1和线程2都会获取锁，但是它们获取的是不同的两把锁。



5. 假定printA和printB都是static synchronized方法。线程1调用printA，线程2调用printB。请问打印结果？**（难）**

```
AAA
BBB
```

当`synchronized`关键字修饰静态方法时，相当于给该方法中所有代码添加锁，并且这个锁来自于该类的`Class`对象。



6. 假定printA和printB是static synchronized方法。有两个手机对象。线程1通过手机1调用printA，线程2通过手机2调用printB。请问打印结果？

```
AAA
BBB
```



7. 假定printB是static synchronized方法。线程1通过对象调用printA，线程2通过对象调用printB。请问打印结果？**（难）**

```
BBB
AAA
```

解释：线程1获取的锁来自`Class`对象，线程2获取的锁来自`this`对象。所以两个线程并没有发送竞争。



8. 假定printA是static synchronized方法，printB是普通同步方法。有两个手机对象。线程1通过手机1调用printA，线程2通过手机2调用printB。请问打印结果。

```
BBB
AAA
```

解释：同上，两条线程获取的不是同一把锁。



## 总结

以上案例主要说明了`synchronized`修饰的普通方法等价于`synchronized`修饰方法中的所有代码，并且通过**this对象**获取锁；而`synchronized`修饰的静态方法等价于`synchronized`修饰方法中的所有代码，并且通过**类的Class对象**获取锁。
