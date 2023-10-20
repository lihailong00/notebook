# JUC——Blocking Queue

[toc]

## 关于阻塞队列

- 队列的核心功能是**先进后出**，从数据结构的角度来看，可以采用数组、链表等方式实现队列。

  补充：优先队列的实现更复杂，它能够保证每次取出的元素都是优先级最高的。其底层原理是**堆**。



- 在多线程环境下，我们还需要考虑**线程安全**、**并发性能**、**队列阻塞策略**等问题。
  - 线程安全
    - 使用锁保证线程安全。
    - 使用volatile关键字来保证数据可见性。
    - 使用CAS（Compare and Swap）操作来实现原子性。
  - 并发性能
    - 非公平锁与公平锁。
    - 减小锁的粒度。
  - 队列阻塞策略（基于生产者-消费者模型理解）
    - 队列空时，**阻塞**消费者；队列满时，**阻塞**生产者。
    - 队列空时，消费者**阻塞一段时间**；队列满时，生产者**阻塞一段时间**。
    - 队列空时，消费者抛出**异常**；队列满时，生产者抛出**异常**。



## 常见api

| 方法         | 抛出异常    | 返回特定值 | 阻塞     | 阻塞特定时间           |
| ------------ | ----------- | ---------- | -------- | ---------------------- |
| 入队         | `add(e)`    | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
| 出队         | `remove()`  | `poll()`   | `take()` | `poll(time, unit)`     |
| 获取队首元素 | `element()` | `peek()`   | 不支持   | 不支持                 |



## 常见队列



| 实现类                  | 功能                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `ArrayBlockingQueue`    | **基于数组的阻塞队列**，使用数组存储数据，并需要指定其长度，所以是一个**有界队列** |
| `LinkedBlockingQueue`   | **基于链表的阻塞队列**，使用链表存储数据，默认是一个**无界队列**；也可以通过构造方法中的`capacity`设置最大元素数量，所以也可以作为**有界队列** |
| `SynchronousQueue`      | 一种没有缓冲的队列，生产者产生的数据直接会被消费者获取并且立刻消费 |
| `PriorityBlockingQueue` | 基于**优先级别的阻塞队列**，底层基于数组实现，是一个**无界队列** |
| `DelayQueue`            | **延迟队列**，其中的元素只有到了其指定的延迟时间，才能够从队列中出队 |





## 使用案例

### `LinkedBlockingQueue`

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.atomic.AtomicInteger;

public class Main {
    static AtomicInteger sell = new AtomicInteger(0);
    static AtomicInteger buy = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        // 创建LinkedBlockingQueue，并设置最大容量为3
        BlockingQueue<Object> blockingQueue = new LinkedBlockingQueue<>(3);

        // 1号生产者
        Thread maker1 = new Thread(() -> {
            // 制作100块饼子
            for (int i = 0; i < 100; i++) {
                try {
                    // put 如果队列已满，则阻塞该线程
                    blockingQueue.put("厨师1 饼子" + i);
                    // 出售数量原子性增加
                    sell.incrementAndGet();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "厨师1");

        // 2号生产者
        Thread maker2 = new Thread(() -> {
            // 制作100块饼子
            for (int i = 0; i < 100; i++) {
                try {
                    // put 如果队列已满，则阻塞该线程
                    blockingQueue.put("厨师2 饼子" + i);
                    // 出售数量原子性增加
                    sell.incrementAndGet();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "厨师2");

        // 1号消费者
        Thread client1 = new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    // take 如果队列为空，则阻塞该线程
                    blockingQueue.take();
                    // 购买数量原子性增加
                    buy.incrementAndGet();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "客户1");

        // 2号消费者
        Thread client2 = new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    // take 如果队列为空，则阻塞该线程
                    blockingQueue.take();
                    // 购买数量原子性增加
                    buy.incrementAndGet();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "客户2");

        // 启动生产者、消费者线程
        maker1.start();
        maker2.start();
        client1.start();
        client2.start();

        // 等待生产者、消费者执行完毕后，继续执行主线程
        maker1.join();
        maker2.join();
        client1.join();
        client2.join();

        System.out.println("卖出饼子: " + sell.get());
        System.out.println("买入饼子: " + buy.get());
    }
}
```



### `SynchronousQueue`

1. `SynchronousQueue`并不是真正意义上的队列，因为它没有容量。

2. 当一个线程尝试向`SynchronousQueue`中插入元素时，它会被阻塞，直到另一个线程从`SynchronousQueue`中取出该元素。

3. 当一个线程尝试从`SynchronousQueue`中取出元素时，它也会被阻塞，直到另一个线程向`SynchronousQueue`中插入一个元素。

   

```java
import java.util.StringJoiner;
import java.util.concurrent.*;

public class Main {
    static void printInfo(String tag) {
        String result = new StringJoiner("\t|\t")
                .add(String.valueOf(System.currentTimeMillis()))
                .add(String.valueOf(Thread.currentThread().getId()))
                .add(Thread.currentThread().getName())
                .add(tag)
                .toString();
        System.out.println(result);
    }

    public static void main(String[] args) throws InterruptedException {
        printInfo("开始执行");
        BlockingQueue<Object> queue = new SynchronousQueue<>();
        new Thread(() -> {
            try {
                queue.put("饼子1");
                printInfo("放入饼子1");
                Thread.sleep(5000);
                queue.put("饼子2");
                printInfo("放入饼子2");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).start();

        Thread.sleep(5000);

        new Thread(() -> {
            try {
                queue.take();
                printInfo("取出饼子1");
                queue.take();
                printInfo("取出饼子2");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }).start();
    }
}
```



> 结果

```
1684077087342	|	1	|	main	|	开始执行
1684077092406	|	12	|	Thread-0	|	放入饼子1
1684077092406	|	13	|	Thread-1	|	取出饼子1
1684077097418	|	12	|	Thread-0	|	放入饼子2
1684077097418	|	13	|	Thread-1	|	取出饼子2
```

