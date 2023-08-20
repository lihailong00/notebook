# JUC——lock

[toc]

可重入：可以多次获取锁



## 重要方法

```java
tryLock();  // 非公平锁

tryLock(long timeout, TimeUnit unit);

lock();  // 获取公平锁，未获取到则一直等待

lockInterruptibly();  // 获取锁的过程中，如果收到中断信号，则会抛出异常

unlock();  // 尝试释放锁
```

