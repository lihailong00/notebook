# CompletableFuture

[toc]



## 是什么

`CompletableFuture`类是Java 8中引入的一个非常强大的异步编程工具。



## 重要函数

`CompletableFuture`类中有几个重要的函数。

- supplyAsync
- thenCompose
- thenCombine



## supplyAsync

`CompletableFuture.supplyAsync()`是Java中`CompletableFuture`类的一个静态方法，它的作用是异步执行一个计算，并返回一个`CompletableFuture`对象，该对象会在计算完成后获取计算结果。它可以用于异步执行一个计算任务，而无需等待该任务的完成。



## thenCompose

将两个异步任务**“串联”**在一起，形成一个新的异步任务。

具体来说，`thenCompose()`方法接受一个参数：

- `fn`：一个`Function`对象，用于将当前`CompletableFuture`的结果作为输入，生成一个新的`CompletableFuture`对象。

```java
import java.util.StringJoiner;
import java.util.concurrent.CompletableFuture;

public class Main {
    /**
     * 线程阻塞一段时间
     * @param millis
     */
    public static void sleepMillis(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 打印自定义内容、当前时间和当前线程id
     * @param tag
     */
    public static void printTimeAndThread(String tag) {
        String result = new StringJoiner("\t|\t")
                .add(String.valueOf(System.currentTimeMillis()))
                .add(String.valueOf(Thread.currentThread().getId()))
                .add(Thread.currentThread().getName())
                .add(tag)
                .toString();
        System.out.println(result);
    }

    /**
     * 场景：
     * 晓龙进入餐厅后，点了一份番茄炒蛋，点完菜后便开始玩B站。
     * 厨师收到晓龙的点菜请求后开始炒菜。
     * 炒完菜后，服务员将菜端上桌。
     * 晓龙开始吃饭。
     * @param args
     */
    public static void main(String[] args) {
        Main.printTimeAndThread("晓龙进入餐厅");
        Main.printTimeAndThread("晓龙点了番茄炒蛋");

        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
            Main.printTimeAndThread("厨师炒番茄");
            Main.sleepMillis(200);
            Main.printTimeAndThread("厨师炒蛋");
            Main.sleepMillis(100);
            return "番茄炒蛋做好了";
        }).thenCompose(dish -> CompletableFuture.supplyAsync(() -> {
            Main.printTimeAndThread(dish);
            Main.sleepMillis(100);
            return "服务员端菜。";
        }));
        
        Main.printTimeAndThread("晓龙在看B站");
        // 主线程阻塞于此，知道cf1对象执行完毕
        Main.printTimeAndThread(String.format("%s, 晓龙开吃", cf1.join()));
    }
}
```



运行结果：

```
1680014725804	|	1	|	main	|	晓龙进入餐厅
1680014725804	|	1	|	main	|	晓龙点了番茄炒蛋
1680014725860	|	12	|	ForkJoinPool.commonPool-worker-1	|	厨师炒番茄
1680014725861	|	1	|	main	|	晓龙在看B站
1680014726073	|	12	|	ForkJoinPool.commonPool-worker-1	|	厨师炒蛋
1680014726185	|	12	|	ForkJoinPool.commonPool-worker-1	|	番茄炒蛋做好了
1680014726324	|	1	|	main	|	服务员端菜。, 晓龙开吃
```





## thenCombine()

`thenCombine()`是Java中`CompletableFuture`类的一种方法，用于组合两个`CompletableFuture`对象的结果，并在它们都完成时执行一个操作。

具体来说，`thenCombine()`方法接受两个参数：

- `other`: 另一个`CompletableFuture`对象，用于与当前对象组合结果。
- `fn`: 一个`BiFunction`对象，用于组合两个`CompletableFuture`对象的结果。



```java
package org.lee._synchronized;

import java.util.StringJoiner;
import java.util.concurrent.CompletableFuture;

public class Main {
    /**
     * 线程阻塞一段时间
     * @param millis
     */
    public static void sleepMillis(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 打印自定义内容、当前时间和当前线程id
     * @param tag
     */
    public static void printTimeAndThread(String tag) {
        String result = new StringJoiner("\t|\t")
                .add(String.valueOf(System.currentTimeMillis()))
                .add(String.valueOf(Thread.currentThread().getId()))
                .add(Thread.currentThread().getName())
                .add(tag)
                .toString();
        System.out.println(result);
    }

    /**
     * 场景：
     * 晓龙进入餐厅后，点了一份番茄炒蛋，点完菜后便开始玩B站。
     * 厨师收到晓龙的点菜请求后开始炒菜，同时服务员开始煮饭。
     * 当炒菜和煮饭同时完成后，服务员将饭和菜都端上桌。
     * @param args
     */
    public static void main(String[] args) {
        Main.printTimeAndThread("晓龙进入餐厅");
        Main.printTimeAndThread("晓龙点了番茄炒蛋");

        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
            Main.printTimeAndThread("厨师炒番茄");
            Main.sleepMillis(200);
            Main.printTimeAndThread("厨师炒蛋");
            Main.sleepMillis(100);
            return "番茄炒蛋做好了";
        }).thenCombine(CompletableFuture.supplyAsync(() -> {
            Main.printTimeAndThread("服务员在煮饭");
            Main.sleepMillis(150);
            return "饭煮好啦";
        }), (dish, rice) -> {  // 两个参数分别是两个CompletableFuture对象返回的参数
            Main.printTimeAndThread(dish);
            Main.printTimeAndThread(rice);
            Main.printTimeAndThread("服务员在端饭菜");
            Main.sleepMillis(100);
            return "服务员端上桌啦！";
        });

        Main.printTimeAndThread("晓龙在看B站");
        // 主线程阻塞于此，知道cf1对象执行完毕
        Main.printTimeAndThread(String.format("%s, 晓龙开吃", cf1.join()));
    }
}
```



运行结果：

```
1680014288444	|	1	|	main	|	晓龙进入餐厅
1680014288444	|	1	|	main	|	晓龙点了番茄炒蛋
1680014288502	|	12	|	ForkJoinPool.commonPool-worker-1	|	厨师炒番茄
1680014288502	|	13	|	ForkJoinPool.commonPool-worker-2	|	服务员在煮饭
1680014288503	|	1	|	main	|	晓龙在看B站
1680014288705	|	12	|	ForkJoinPool.commonPool-worker-1	|	厨师炒蛋
1680014288815	|	12	|	ForkJoinPool.commonPool-worker-1	|	番茄炒蛋做好了
1680014288815	|	12	|	ForkJoinPool.commonPool-worker-1	|	饭煮好啦
1680014288815	|	12	|	ForkJoinPool.commonPool-worker-1	|	服务员在端饭菜
1680014288945	|	1	|	main	|	服务员端上桌啦！, 晓龙开吃
```

