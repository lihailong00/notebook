# AQS

[toc]



## AQS是什么

`AQS`是一个`JUC`包里核心的抽象类，开发者可以继承`AQS`类实现自定义同步器类，从而更方便实现各种同步机制，例如锁、信号量、倒计时器。

`AQS`的核心思想是基于一个**双向链表队列**来实现同步。



## 多线程并发的挑战以及解决方案

当多个线程同时访问相同的共享资源时，会遇到以下问题：

1. 哪个/哪些线程可以获得该共享资源？
2. 没有获得共享资源的线程该怎么办？
3. 使用完共享资源后，该如何通知别的线程？



下面我举一个案例：多个人同时需要争抢一个厕所。

![混乱局面](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/698fb76afabab596a060d2ca91209a59.png)

如果让这些人盲目争抢厕所，可能会非常混乱。

按照我们朴素的思想来看，应该让这些人排成一队，然后按照先来后到的方式有序使用厕所。

![有序排队](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/92a605939f8b74b4151ff286a88c4375.png)



AQS的思想就是这样！AQS基于双向链表队列管理线程。

在使用AQS的使用，通常会让自定义的同步器类`extends`AQS抽象类。然后创建一个自定义同步器对象。一个同步器对象对一个共享资源负责。

那么我们来看看同步器对象中封装了哪些东西？同步器对象又是如何有序管理不同线程访问共享资源的？

![同步器对象](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/20668e51e687203b1e913df38c4e5499.png)

可以看到，同步器对象中除了队列以外，还有一个`state`对象和一个`exclusiveOwnerThread`对象。

state：不同的子类中会有不同的含义，这里不讲。

exclusiveOwnerThread：记录当前资源被哪个线程占有。



## 资源分配和释放的完整过程

下面演示一个新人想使用厕所的完整过程。



当新人刚尝试使用厕所时：

![新加入线程](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/341a40e14ff241e74df62532a394620c.png)





当新人排队时：

![排队](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/f73c8d9a2127348c28ca9d56f644768e.png)



![尝试进入厕所](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/6b532048701865032fff6fa82d4eed5d.png)





当前一个人上完厕所后，会**唤醒**下一个人使用资源。

![释放资源](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/2f986ad19d1e1563bc9920964ed8fcf7.png)



## AQS核心方法

```java
tryAcquire(int);        // 尝试获取独占锁，可获取返回true，否则false
tryRelease(int);        // 尝试释放独占锁，可释放返回true，否则false
tryAcquireShared(int);  // 尝试以共享方式获取锁，失败返回负数，只能获取一次返回0，否则返回个数
tryReleaseShared(int);  // 尝试释放共享锁，可获取返回true，否则false
isHeldExclusively();    // 判断线程是否独占资源
```
