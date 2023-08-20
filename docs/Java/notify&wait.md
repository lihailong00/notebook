# notify&wait

[toc]



## 

必须在同步方法或同步代码块中使用。

每个对象都有notify函数和wait函数。

wait会阻塞线程并释放对象的锁。wait函数好像也可以传入时间参数。

notify调用后需要等同步代码块执行完才会唤醒线程。

notifyall会唤醒该对象的等待队列中，所有的线程。

实现一个生产者消费者模型。升级：如果运行多个消费者线程运行，该怎么设置条件变量呢？



#### [1115. 交替打印 FooBar](https://leetcode.cn/problems/print-foobar-alternately/)

synchronized + wait + notify

yield

