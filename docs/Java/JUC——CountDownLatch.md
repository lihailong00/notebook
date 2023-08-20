# JUC——CountDownLatch

[toc]



## 是什么

`CountDownLatch`是`JUC`包下的一个工具类，用于**控制线程的等待和并发执行**。

举个例子，我可以让5个线程全部执行完毕后，再执行接下来的代码。



## 代码

下面是捕获一尾至九尾，全部捕获完毕后合成十尾的过程。

```java
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;

public class Main {
    static AtomicInteger i = new AtomicInteger(0);
    public static void main(String[] args) {
        CountDownLatch countDownLatch = new CountDownLatch(9);
        for (int j = 0; j < 9; j++) {
            new Thread(() -> {
                int timeSpend = new Random().nextInt(10);
                try {
                    Thread.sleep(timeSpend * 1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println("已捕获" + i.incrementAndGet() + "尾");
                countDownLatch.countDown();
            }).start();
        }

        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        System.out.println("捕获完毕，合并十尾！");
    }
}
```

