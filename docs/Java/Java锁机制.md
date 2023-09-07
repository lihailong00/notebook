# Java锁机制

[toc]



## 概述

- 锁是实现线程同步的重要工具，保证并发访问共享资源时，共享资源满足一致性。
- 可以按照不同的维度对“锁”进行分类。
- Java中，一个锁存放在一个对象的**对象头**中。





## Java对象的结构

Java对象主要包含3个部分。

1. 对象头。
   1. Mark Word：存放当前对象的状态信息。
   2. Klass Pointer：是一个指针，指向当前对象类型对应在方法区中的类数据。
2. 实例数据。
3. 对齐填充数据：保证Java对象的大小是8Byte的整数倍。



## 乐观锁 VS 悲观锁

1. 悲观锁：

   - 并发访问共享资源时，悲观锁认为自己占有共享资源时，别的线程会使用共享资源。因此获取资源时先加锁，保证共享资源不被其他线程修改。
   - Java中，`synchronized`和`Lock`都是基于悲观锁实现的。
   - 悲观锁适合写多读少的场景。

2. 乐观锁：

   - 并发访问共享资源时，乐观锁认为自己占有共享资源时，别的线程不会使用该共享资源，因此获取资源时不用加锁。
   - 在更新共享资源时判断共享资源是否被更新，如果资源没有被修改，则直接更新资源；如果资源被修改，则执行其他操作（比如自旋或报错）
   - Java中，采用无锁的方式实现乐观锁，最常采用CAS算法。其中`AtomicInteger`就是基于CAS。
   - 乐观锁适合读多写少的场景。

3. 乐观锁与悲观锁的代码对比：

   ```java
   // ============乐观锁============
   private Lock lock = new ReentrantLock();
   public void getResourse() {
       lock.lock();
     	// 访问共享资源
       lock.unlock();
   }
   
   // ============悲观锁============
   private AtomicInteger atomicInteger = new AtomicInteger();
   public void getResourse() {
       // 直接修改共享资源
       atomicInteger.incrementAndGet();
   }
   ```





## synchronized的四种锁

`synchronized`对应了4种锁：无锁、偏向锁、轻量级锁和重量级锁，这四种锁分别对应了对象头中Mark Word的四种状态。



## 重量级锁

重量级锁基于monitor实现。

**我们可以简单将monitor比作一个空间，同一时间只能进入一个线程。当一个线程获取锁时，该线程进入moniter，此时其他线程无法进入moniter。当该线程释放锁时，该线程退出monitor，其他线程才有机会获得锁。**

**锁和moniter一一对应。**

下面用代码的方式演示原理：

```java
public class Main {
    static int a = 0;
    public static void main(String[] args) {
        for (int i = 1; i <= 10000; i++) {
            synchronized (Main.class) {
                a++;
            }
        }
    }
}
```

上述代码通过编译和反编译后（`javac Main.java`和`javap -c Main.class`），可以得到以下代码。

```java
Compiled from "Main.java"
public class Main {
  static int a;

  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: iconst_1
       1: istore_1
       2: iload_1
       3: sipush        10000
       6: if_icmpgt     38
       9: ldc           #2                  // class Main
      11: dup
      12: astore_2
      13: monitorenter
      14: getstatic     #3                  // Field a:I
      17: iconst_1
      18: iadd
      19: putstatic     #3                  // Field a:I
      22: aload_2
      23: monitorexit
      24: goto          32
      27: astore_3
      28: aload_2
      29: monitorexit
      30: aload_3
      31: athrow
      32: iinc          1, 1
      35: goto          2
      38: return
    Exception table:
       from    to  target type
          14    24    27   any
          27    30    27   any

  static {};
    Code:
       0: iconst_0
       1: putstatic     #3                  // Field a:I
       4: return
}

```

其中第13行和第23行分别对应线程进入moniter和线程离开moniter。



### 轻量级锁

若**多个线程在不同时刻访问共享资源**，则不会发生竞争关系。如果线程切换仍使用moniter，就会损耗性能。

轻量级锁并不能解决线程冲突问题，当共享资源被占用时，另一个线程访问共享资源，就会发生锁升级（轻量级锁->重量级锁）。





### 偏向锁（OpenJDK15已废弃）

大多数情况下，只有**一个线程会访问共享资源**。因此我们想到在对象的MarkWord中记录线程ID，当下一次相同的线程访问共享资源时，就不采用moniter，而是直接获取资源。



废弃原因：

原先的集合类（Vector，HashTable...）大量使用到`synchonized`保证线程安全。但如今，更多场景使用无锁的集合类（ArrayList，HashMap），即便是并发场景，也使用ConcurrentHashMap这种高性能集合。因此偏向锁带来的收益越来越少。

偏向锁增加了系统的复杂度。



### 无锁

没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。

无锁的实现方式之一是CAS算法。

无锁无法全面代替有锁，但无锁在某些场合下的性能是非常高的。





## 公平锁 VS 非公平锁

我们用一个排队买东西的场景解释这两种锁。

公平锁：一些人正排队买瓜，此时你走过来买瓜。你肯定会**直接排到队尾**（因为年轻人的素质大多数都挺高的），这样对大家都公平。

非公平锁：此时来了一个大妈，她直接**插队到最前面**买瓜，如果她插队成功，则她先买瓜；如果插队失败，才会拍到队尾。大妈这种插队的行为不公平！
