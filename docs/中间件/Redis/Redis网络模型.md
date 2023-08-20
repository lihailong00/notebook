# Redis网络模型

[toc]

## 用户空间和内核空间

1. 操作系统将内存分为用户空间和内核空间，目的是为了安全性等。

2. 用户空间是指应用程序运行的内存区域；内核空间是操作系统的核心部分。

3. Linux系统为了提高IO效率，会在用户空间和内核空间都添加缓冲区。
   1. 写数据时，要把用户缓冲数据拷贝到内核缓冲区中，然后写入设备。
   2. 读数据时，要从设备读取数据到内核缓冲区，然后拷贝到用户缓冲区。



## 五种IO模型

Linux下I/O的过程是：用户空间的用户缓冲区等待内核空间的内核缓冲区就绪；内核缓冲区将数据发送给用户缓冲区。针对这两步过程，《Unix网络编程》给出了五种实现策略。

### 阻塞IO

BIO模型中，每一个连接都需要一个线程处理。

用户的请求会导致线程不断销毁和创建，浪费大量资源，不过可以采用线程池的方式减少线程的创建销毁次数。

但是当某一时刻连接过多时，会占用大量线程。

从内核的角度分析，当应用程序发起一次`recvfrom`系统调用时，内核便开始准备数据，同时用户进程/线程被阻塞。当数据准备就绪后，内核将数据从内核空间的缓冲区复制到用户空间的缓冲区，并将结果返回给应用程序，之后应用进程继续执行。

![在这里插入图片描述](https://codeantenna.com/image/https://img-blog.csdnimg.cn/20200331220211779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9fbWFfbm9uZ19sYXN0,size_16,color_FFFFFF,t_70#pic_center)





### 非阻塞IO

当用户进程发出`recvfrom`操作时，如果内核中的数据还没有准备好，那么内核并不会阻塞用户进程，而是立刻返回结果。从用户进程角度讲 ，它发起一个`recvfrom`操作后，并不需要等待，而是马上就得到了一个结果，用户进程通过结果判断数据是否准备好，如果没有准备好过段时间再次发送`recvfrom`操作，一旦kernel中的数据准备好了，并且又再次收到了用户进程的`recvfrom`调用操作，马上将数据从内核缓冲区拷贝到了用户进程缓冲区。

![在这里插入图片描述](https://codeantenna.com/image/https://img-blog.csdnimg.cn/20200331220233622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9fbWFfbm9uZ19sYXN0,size_16,color_FFFFFF,t_70#pic_center)

NIO用户进程总是发起`recvfrom`反而会浪费资源。感觉没有BIO高效。

**只有当NIO和IO多路复用等机制一起使用时，才会发出它的威力。**



提个简单的问题：如果只有一个连接，那么NIO和BIO谁更高效？

显然是BIO更高效，在只有一个连接的情况下，BIO模型中用户进程只需要向内核发送一次请求，然后等待数据的到来即可；而NIO需要不断的轮寻。

### IO多路复用

I/O多路复用是指通过一种机制，使得**一个进程可以监视多个文件描述符**（包括socket连接、标准输入输出、文件等），一旦某个文件描述符就绪（一般是读写就绪），则可以通知进程进行相应的读写操作。

而监听FD的方式又有多种，此时就衍生出了三种模型：select、poll、epoll。

#### select

select是最古老的I/O多路复用机制，它使用一个`fd_set`类型的数据结构（数组）来管理多个文件描述符，可以监视读、写和错误三种事件。

```c
// fd_set 结构
typedef struct fd_set {
  uint32_t fds_bits[FD_SETSIZE / 32];
} fd_set;
```

简单的过程是这样的：应用程序的一条进程/线程监听一个存放文件描述符的数组，并且不断遍历数组。我们默认通过文件描述符可以获取文件的状态。当用户进程发现有事件（比如一个连接）需要处理时，就处理该事件。

完整的过程是这样的：用户调用select时，内核会将`fd_set`对象从用户空间拷贝到内核空间，并在内核空间遍历整个文件描述符集合（内核通过fd可以找到文件的状态）。当存在事件就绪时，select函数就会返回就绪事件的个数，并且把`fs_set`数组从内核空间拷贝会用户空间。用户进程再遍历一次`fd_set`数组，从而获得就绪的文件描述符，并进行操作。



问题：select调用中，如果只有一个文件，它的fd是1000，那么我的fd_set的大小是多少呢？

答案：fd_set数组的大小是1000位，尽管这会浪费很多资源。

![在这里插入图片描述](https://codeantenna.com/image/https://img-blog.csdnimg.cn/20200331220251317.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9fbWFfbm9uZ19sYXN0,size_16,color_FFFFFF,t_70#pic_center)



#### poll

和select模型几乎一样。唯一不同的是poll模型中内核空间的fd集合是链表（用户空间的fd集合还是数组），理论上可以存放无限个fd。而select模型中使用`fd_set`类型的数组存放fd，而数组大小最大值规定为1024。



#### epoll

1. 用户调用`epoll_create(int size)`函数，会在**内核**创建一个eventpoll结构体，并返回结构体对象的句柄epfd。
2. 用户再调用`epoll_ctl`，即可将fd加入eventpoll对象的红黑树中，并设置ep_poll_callback函数。当callback触发时，将红黑树中对应的fd加入就绪列表（链表）中。
3. 用户调用`epoll_wait`，内核会检查就绪列表（而不是红黑树），并只会将就绪的fd拷贝到用户空间的一个空数组中。

epoll减少了fd集合的拷贝次数和拷贝数量，减少了遍历fd元素的次数。



epoll的结构体：

```c++
struct eventpoll {
    // ...
    struct rb_root rbr;  // 一颗红黑树，记录要监听的fd
    struct list_head rdlist;  // 一个链表，记录就绪的fd
    // ...
}

```



### 信号驱动IO



### 异步IO
