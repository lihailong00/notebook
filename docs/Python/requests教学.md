# requests教学

[toc]



## 前言

本文主要围绕如何`requests`库的使用方法展开讲解。

`requests`库可以用于发送`HTTP`请求。



## 主要API

**建议**：使用`session`对象发送信息，而不是使用`requests`对象发送信息。

`session`对象可以保存**cookies**、头部信息，以便在多次访问中不用重复更新cookie。

`session`对象获取方式：`session = requests.session()`。



> 请求函数

主要由两个函数可以用于发送请求：`session.get()`和`session.post()`，也可以使用`requests.get()`和`requests.post()`。



> **关于Cookie**

建议使用`session`对象，这样就不用维护更新`cookie`。



如果使用`requests`对象，那么需要维护`cookie`信息，注意以下问题：

一定不要把Cookie转成字符串，否则会出现细碎的问题。使用Cookie时，直接使用`RequestsCookieJar`对象。

```python
import requests
from requests.cookies import RequestsCookieJar


# 首先创建cookie对象。一个Cookie对象中可以存放多条Cookie信息。
cookies = RequestsCookieJar()
cookies.set('name', 'lhl')
resp = requests.get(url='http://localhost:8080', cookies=cookies)
print(resp.text)  # 得到 name=lhl
```



基于Spring Boot的服务器部分代码

```java
package com.lee.cookiedemo.controller;

import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping
public class IndexController {
    @RequestMapping
    public String get(@CookieValue("name") String name) {
        return "name=" + name;
    }
}
```



> 是否验证SSL证书

`session.get(verify=False)`。默认值为True，验证证书可以防止中间人攻击。

通常情况下，我习惯设定为`False`。因为很多网站的`SSL`证书无法通过验证（比如比较垃圾的教务处网站）。



> 自动重定向

`session.get(allow_redirects=True)`。默认值为true。

当访问某个URL（例如`localhost:8080/redirect`）时，服务器可能会返回一个**重定向报文**，客户端接收到信息后，会访问重定向报文中的地址。



重定向报文需要满足如下格式：

1. 状态码是302.
2. HTTP的headers中有`Location`字段。



Pyrhon的requests库中，当发出一个http请求后，如果接收到重定向报文，默认会访问重定向目标地址。如果将`allow_redirects`属性设置为`False`，则不会发生跳转。



测试案例：

服务器代码（Spring Boot）

```java
package com.lee.cookiedemo.controller;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping
public class IndexController {
    @RequestMapping("redirect")
    public ResponseEntity<Void> redirect() {
        // 设置重定向的URL
        String redirectUrl = "http://example.com";

        // 创建一个 ResponseEntity，并设置状态码和重定向的URL
        ResponseEntity<Void> responseEntity = ResponseEntity.status(HttpStatus.FOUND)
                .header("Location", redirectUrl)
                .build();

        // 返回 ResponseEntity 对象
        return responseEntity;
    }
}
```



爬虫代码

```python
import requests

# 默认允许重定向
resp = requests.get(url='http://localhost:8080/redirect')
print('text=\n' + resp.text)

# 禁用重定向
resp = requests.get(url='http://localhost:8080/redirect', allow_redirects=False)
print('text=\n' + resp.text)
```



> 乱码问题

当我们爬取一个html文件后，有时会发现html文件中的中文乱码。原因是在响应的headers中，没有指明数据的编码方式。此时Python会采用默认的`ISO-8859-1`编码方式，从而导致中文乱码。

解决方式是指定http响应的编码方式：`resp.encoding = 'utf-8'`。

案例如下：

```python
import requests

resp = requests.get('https://www.baidu.com')  # 使用前关闭代理
print(resp.encoding)
print(resp.text)
resp.encoding = 'utf-8'
print(resp.text)
```



有时候爬取的html文件没有乱码，但用浏览器打开后乱码。可能是浏览器采用的错误的解码方式解码数据。此时我们只需要在html文件中加入：`<meta charset="UTF-8">`即可。
