# JUC——ReentrantLock

[toc]



## 特性概览

ReentrantLock意思为可重入锁，指的是一个线程能够多次访问同一个临界资源。“可重入”也就是多次进入临界资源的意思。



## ReentrantLock和Synchronized

|            | ReentrantLock                  | Synchronized     |
| :--------- | :----------------------------- | ---------------- |
| 锁实现机制 | 依赖AQS                        | 监视器模式       |
| 灵活性     | 支持响应中断、超时、尝试获取锁 | 不灵活           |
| 释放形式   | 必须显示调用unlock()释放锁     | 自动释放监视器   |
| 锁类型     | 公平锁&非公平锁                | 非公平锁         |
| 条件队列   | 可关联多个条件队列             | 关联一个条件队列 |
| 可重入性   | 可重入                         | 可重入           |



## 重要方法

```java
tryLock();  // 非公平锁

tryLock(long timeout, TimeUnit unit);

lock();  // 获取公平锁，未获取到则一直等待

lockInterruptibly();  // 获取锁的过程中，如果收到中断信号，则会抛出异常

unlock();  // 尝试释放锁
```

