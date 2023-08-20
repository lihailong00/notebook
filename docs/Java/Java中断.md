# Java中断

[toc]



## 什么是中断

> 易错：不要误认为运行态的线程接收到中断信号后会停止执行！



1. **中断**通常是指**中断信号**，中断信号由一个发送者线程（`Runnable`状态）发送给接收者线程（`BLOCK/WAITING/TIMED_WAITING`状态），接收者线程会被**唤醒**并**抛出**一个**异常**（`InterruptedException`），并将中断标志位设为`true`。

   > 思考：阻塞线程收到中断信号后会被唤醒，为什么要抛出`InterruptedException`呢？
   >
   > 答案：为了知道线程为什么被唤醒。



2. 关于中断标志位：
   - 每一个线程都有一个中断标志位（`true/false`），且线程也能设置其他线程的中断标志位。
   - 可以通过`Thread.currentThread().isInterrupted()`函数获取当前线程的标志位。



## 发送中断信号的案例

常用函数：

`Thread.currentThread().isInterrupted()`：查看线程的中断标志位。

`Thread.interrupted()`：查看线程的中断标志位， 清除中断标志位



```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("t1线程正常执行完毕。");
            } catch (InterruptedException e) {
                System.out.println("t1线程收到中断信号，正在回收资源...");

                // 为了让主线程捕获t1线程的RUNNABLE状态
                for (int i = 0; i < 100000000; i++) {}
            }
        });

        // 启动t1线程
        t1.start();

        // 主线程休眠1ms
        Thread.sleep(1);

        // t1线程正处于TIMED_WAITING状态
        System.out.println("t1线程当前状态: " + t1.getState());

        // 此时t1线程还未收到中断信号，所以值为false
        System.out.println("t1线程中断标志位: " + t1.isInterrupted());

        // 向t1线程发送中断信号
        t1.interrupt();

        // 主线程休眠1ms
        Thread.sleep(1);

        // 此时t1线程已经收到中断信号，因此t1线程的标志位值为true
        System.out.println("t1线程中断标志位: " + t1.isInterrupted());

        // 因为t1线程收到中断信号后被唤醒且正在执行，所以t1线程的状态是RUNNABLE
        System.out.println("t1线程当前状态: " + t1.getState());

        Thread.sleep(100);
        
        // 主线程线程阻塞100ms，确保t1线程执行完毕
        System.out.println("t1线程当前状态: " + t1.getState());
    }
}
```





线程在runnable状态下收到中断信号会怎样？发送方来自自己或别人。

直接忽略中断信号。

如果想处理中断信号该怎么做呢？

区别是什么？

```
Thread.currentThread().isInterrupted();  只是查看
Thread.interrupted();  清除中断标志位
```


