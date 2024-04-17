# JWT的坏处



## 参考文章

https://mp.weixin.qq.com/s/7F_zrBnbKVktrIo2PSxgHw



## 前置知识

1. 无状态JWT：JWT中存用户信息，Session数据直接编码到JWT中。
2. 有状态JWT：JWT中通常存Session引用（或SessionId），Session数据存放在服务端。



## JWT的优势并不明显

1. **JWT易于水平拓展**：无状态的JWT才易于水平拓展。因为新增服务器时，服务器不会存放任何JWT Token；对于Session机制，如果新增服务器，需要保证在这个服务器中有用户Session信息才行。

> 然而我们可以用一个Redis存放一个服务器集群用到的Session信息。并且绝大多数情况下，后端项目都是单体的，用户量很小。



2. 



## JWT的缺点

1. 无法直接废弃，当然我们可以在Redis中存放token达到这种效果，但是这和普通的Session机制有什么区别呢？
2. 无法直接续签。传统的Cookie续签方案几乎所有框架都自带，而JWT续签需要通过麻烦的操作（比如OAuth2.0的双token机制）。





## JWT的适用场景

1. 给用户发一封邮件用于激活账号，30分钟内有效。
