# Java垃圾回收

[toc]

## 引言

谈论到垃圾回收，我们需要知道以下问题：

1. 如何判断一个对象是“垃圾”？
2. 如何清理垃圾？换种说法，如何释放内存块？



## 死亡对象判断方法

### 引用计数法

实现简单，但是不能解决循环依赖的问题。



### 可达性分析算法

有一类点，成为**GC-ROOT**，以每一个GC-ROOT为树根向下发散，判断哪些对象可达。如果某个对象不能通过GC-ROOT抵达，那么它容易被回收。

那么哪些对象可以作为 GC Roots 呢？

- **虚拟机栈(栈帧中的本地变量表)中引用的对象**
- 本地方法栈(Native 方法)中引用的对象
- **方法区中类静态属性引用的对象**
- 方法区中常量引用的对象
- **所有被同步锁持有的对象**



## 垃圾回收算法

垃圾回收算法主要解决了发现垃圾后，如何回收垃圾。



### 标记-清除 算法

思想：哪些内存区域有垃圾，就标注哪些内存区域。

好处：实现容易。

缺点：容易产生碎片。

![在这里插入图片描述](https://img-blog.csdnimg.cn/cbbb1a4a6b2348638f98e8653cc2a949.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6Imv54y_5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16)



### 标记-复制 算法

思想：将内存分为大小相同的两块。当其中一块用完后，将该内存中存活的对象拷贝到另一块内存并连续存放。

好处：相较于标记-清除算法来说，减少了碎片的产生。

坏处：需要两倍的内存空间。



### 标记-整理 算法

思想：标记垃圾。将存活的对象向前移动。直接清除掉边界意外的内存。

好处：节约内存空间，减少了碎片的产生。

坏处：需要频繁修改对象的位置。

![在这里插入图片描述](https://img-blog.csdnimg.cn/81d7ba5bdb554092af880afe61fec984.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6Imv54y_5ZWK,size_20,color_FFFFFF,t_70,g_se,x_16)





### 分代收集 算法

将Java的堆区分为新生代和老年代。

新生代有大量的对象寿命短，容易死去。我们可以采用标记-复制算法。这样只需要付出一部分内存的代价，就能较好的完成垃圾收集。

老年代对象存活久，可以选择标记-清除或标记-整理算法进行垃圾收集。



## 垃圾回收器

垃圾回收器是一个软件，运行在JVM中。不同的垃圾回收器适用于不同的堆内存，采用了不同的垃圾回收算法。

![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/C9C7B8863B03F1F862C509A17051D945.png)





### CMS垃圾回收器

CMS(Concurrent Mark Swap)，并发标记清除。作用于**老年代**，降低STW。



并发是指垃圾回收线程可以和用户线程同时执行。



CMS垃圾回收器大致分为以下四个步骤：

初始标记：标记`GC root`以及`GC root`可以直接引用的对象，通常速度较快，会产生STW。

并发标记：和用户线程并发执行，标记老年代中的所有对象。

重新标记：

并发清除：采用标记-清除算法，清除老年代中的垃圾。由于不需要移动对象，因此可以多线程。



> 优点

STW时间短。（仍然有STW）



> 缺点：

1. 并发执行垃圾回收线程，会占用较多资源。
2. 无法清除浮动垃圾。并发执行垃圾清除的过程中，用户可能会产生新的垃圾。因此不能等到老年代满了才清除垃圾。
3. 标记-清除算法有内存碎片问题。



### G1垃圾回收器

> 重要特征

1. **全年代**垃圾回收器。
2. 堆内存被切分为一个个大小相同的**内存块**，每一个内存块属于新生代或老年代。所以新生代或老年代的内存不一定连续。
3. 每次回收一部分region块，不用全部回收。
4. region的年龄是动态变化的，一会儿是新生代，一会儿是老年代。
5. 一个region块内，采用标记-复制算法。
6. G1可以**指定STW的时间**，标记-复制清理垃圾的过程中它会选择有价值的region进行回收，从而保证STW的时间小于指定值。





## 内存分配原则

新生代：老年代 = 1:2

eden:from:to=8:1:1
