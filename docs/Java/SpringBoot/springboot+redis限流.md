# springboot+redis对api实现计数器方式的限流

[toc]

## 限流的是什么

限流是限制到达系统的并发请求数量，保证系统能够正常响应部分用户请求，而对于超过限制的流量，则通过拒绝服务的方式保证整体系统的可用性。



## 限流的范围

单机限流和分布式限流。我常用的是单机限流。



## 常用的限流方式

### 计数器

在一段时间内，规定某个 ip/用户/区域 访问 服务器/接口 次数不能超过一个数。

优点：实现简单。

缺点：

### 滑动窗口



### 漏桶



### 令牌桶



## 

