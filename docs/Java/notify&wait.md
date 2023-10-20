# notify&wait

[toc]



## 使用注意事项

1. 必须在同步方法或同步代码块中使用。
2. `notify`函数和`wait`函数成对出现。
3. `wait`会阻塞线程并释放对象的锁。
4. `notify`调用后需要等同步代码块执行完才会唤醒线程。
5. `notifyall`会唤醒该对象的等待队列中所有的线程。

实现一个生产者消费者模型。升级：如果运行多个消费者线程运行，该怎么设置条件变量呢？



## 实战案例

[1115. 交替打印 FooBar](https://leetcode.cn/problems/print-foobar-alternately/)

思路：

代码：

```java
```



