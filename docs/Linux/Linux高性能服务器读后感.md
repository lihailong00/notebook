# Linux高性能服务器读后感

[toc]



## 其他

[参考文档](https://www.cnblogs.com/fuchongjundream/p/3914696.html)

问题：在Unix系统中，一切东西都可以抽象为文件吗？

答案：是滴！



问题：什么是socket？

答案：socket即是一种特殊的文件，一些socket函数就是对其进行的操作（读/写IO、打开、关闭）。



## 第5章

问题：网络序和主机序如何对应大端序和小端序呢？

答案：网络序是大端序，主机序通常是小端序。



问题：主机序和网络序相互转换的api？

答案：

```cpp
#include <netinet/in.h>
htonl();  // host to net long
htons();  // host to net short
ntohl();  // net to host long
ntohs();  // net to host short
```



问题：socket通信涉及到两方，如何表示他们的地址呢？

答案：使用socket地址表示他们。



问题：协议簇是什么？我们通常使用哪个协议簇呢？

答案：协议簇是一大堆协议的集合；通常使用TCP/IP协议簇，里面包括了TCP，UDP，DNS...等协议。



问题：TCP/IP协议簇的socket地址结构是怎样的？（了解即可，需要时查就行）

答案：

```c++
struct sockaddr_in {
    sa_family_t sin_family;  // 地址族，通常设为AF_INET
    u_int16_t sin_port;  // 端口号，用网络字节序表示
    struct in_addr sin_addr;  // ipv4结构体
};
// ipv4结构体
struct in_addr {
    u_int32_t s_addr;  // ipv4地址，要用网络字节序表示
};
```



问题：ip地址和点分十进制如何相互转换？（需要时搜就行）

答案：





问题：如何创建socket？（重要，需要的时候直接抄）

答案：

```cpp
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
// 如果domain为PF_INET，则使用IPv4协议（常用）；如果为PF_INET6，则使用IPv6协议（不常用）。
// 如果type为SOCK_STREAM，则使用tcp协议；如果type为SOCK_UGRAM，则使用UDP协议。
// ??? type为SOCK_NONBLOCK和SOCK_CLOEXEC
```



问题：如何给socket设置地址？

答案：

```cpp
#include <sys/types.h>
#include <sys/socket.h>
int bind(int sockfd, const struct sockaddr* my_addr, socklen_t addrlen);
```

成功返回0，失败返回-1。



问题：如何监听socket？

答案：

```cpp
#include <sys/socket.h>
int listen(int sockfd, int backlog);
// sockfd指被监听的socket，backlog指监听队列的最大长度
```

