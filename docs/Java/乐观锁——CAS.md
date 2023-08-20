# 乐观锁——CAS

[toc]



## 有什么用

在多线程环境下，对共享资源进行读写操作。

因为它没使用锁，所以减少了线程切换的开销，在竞争不激烈的情况下效率较高。

总结：**CAS机制是一种数据更新方式。它是原子操作。操作过程是比较+交换。**



## 场景分析

假定我们有一个资源对象`obj`，`obj`对象中有一个状态值`state`。当`state=0`时，表示资源空闲；当`state=1`时，表示资源暂时不可用。

一条线程在使用资源时，需要先查看资源是否可用，只有`state=0`时才能使用资源；当使用完资源时，将重置`state=0`。



为了实现上述操作，我们有两种实现方式：

- 有锁编程（悲观锁）：给资源上一把锁，某一线程需要使用资源时，先获取这把锁。如果获取成功，就能使用资源；如果获取失败，则该线程阻塞在此处。当某一个线程使用完资源后，释放锁并唤醒其他线程。

  

- 无锁编程（乐观锁）：某一线程需要使用资源时，通过`CAS(Compare And Swap)`，也即比较`obj`对象的`state`值是否与目标值`0`相等，如果相等，则设置`state=1`；如果不相同，则自旋一次后重新比较。通常会设定一个自旋次数，防止一直自旋导致死锁。

  比较+设置`state`这两步是原子操作！

  

  > 有锁编程案例：实现自增（运行时长：13011ms）

  ```java
  public class Main {
      static int num = 0;
      public static void main(String[] args) {
          int max = 100000000;
          Thread t1 = new Thread(() -> {
              for (int i = 0; i < max; i++) {
                  synchronized (Main.class) {
                      num++;
                  }
              }
          });
          Thread t2 = new Thread(() -> {
              for (int i = 0; i < max; i++) {
                  synchronized (Main.class) {
                      num++;
                  }
              }
          });
          long start = System.currentTimeMillis();
          t1.start();
          t2.start();
          try {
              t1.join();
              t2.join();
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          } finally {
              long end = System.currentTimeMillis();
              System.out.println("运行时长: " + (end - start) + "ms.");
              System.out.println("num=" + num);
          }
      }
  }
  ```

  

  > 无锁编程案例（运行时长：2097ms）

  ```java
  import java.util.concurrent.atomic.AtomicInteger;
  
  public class Main {
      static AtomicInteger num = new AtomicInteger(0);
      public static void main(String[] args) {
          int max = 100000000;
          Thread t1 = new Thread(() -> {
              for (int i = 0; i < max; i++) {
                  num.incrementAndGet();
              }
          });
          Thread t2 = new Thread(() -> {
              for (int i = 0; i < max; i++) {
                  num.incrementAndGet();
              }
          });
          long start = System.currentTimeMillis();
          t1.start();
          t2.start();
          try {
              t1.join();
              t2.join();
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          } finally {
              long end = System.currentTimeMillis();
              System.out.println("运行时长: " + (end - start) + "ms.");
              System.out.println("num=" + num);
          }
      }
  }
  ```

  

  > 不做同步操作（运行时长：16ms）
  >
  > 不管对不对，你就说快不快吧:dog:

  ```java
  public class Main {
      static int num = 0;
      public static void main(String[] args) {
          int max = 100000000;
          Thread t1 = new Thread(() -> {
              for (int i = 0; i < max; i++) {
                  num++;
              }
          });
          Thread t2 = new Thread(() -> {
              for (int i = 0; i < max; i++) {
                  num++;
              }
          });
          long start = System.currentTimeMillis();
          t1.start();
          t2.start();
          try {
              t1.join();
              t2.join();
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          } finally {
              long end = System.currentTimeMillis();
              System.out.println("运行时长: " + (end - start) + "ms.");
              System.out.println("num=" + num);
          }
      }
  }
  ```

  

## CAS缺点

1. 没有解决**ABA问题**。
2. 只能保证某一行代码是原子操作，不能保证某个代码块是原子操作，也即**范围不可灵活控制**。
3. 如果没有设置适当的自旋次数，可能会占用大量资源。



### 解决ABA问题：版本号机制

ABA问题
