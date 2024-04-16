# Session和Cookie实战

[toc]



## Session是什么

Session是存放在服务器中的对象。每一个客户端访问服务器时，服务器都会创建一个Session对象记录该用户的访问记录。



在Spring中，调用HttpServletRequest对象的getSession()函数时，如果当前请求中不存在会话（session），则会自动创建一个新的会话，并在响应头中添加一个Set-Cookie头，用于在客户端保存会话标识符（Session ID）。这是Servlet规范的默认行为，Spring Boot并没有改变这个行为。

这是因为在Java Web应用程序中，会话（session）是用于跟踪用户在网站上的活动状态的机制。每个会话都有一个唯一的标识符（Session ID），在客户端和服务器之间来回传递，以便维护会话状态。当客户端第一次请求应用程序时，服务器会在响应头中添加一个Set-Cookie头，用于将会话标识符发送给客户端。客户端会将该标识符存储在Cookie中，并在后续请求中自动发送给服务器，以便维护会话状态。



## Cookie是什么

服务器创建session对象后，会将sessionid放入cookie对象，并将cookie对象返回给用户。用户在之后的访问过程中只需要携带cookie对象即可。



## Session的创建过程

当

具体来说当服务器收到一个新的连接时，会在响应中包含`Set-Cookie`字段，客户端浏览器收到后会将`Set-Cookie`中的信息（也叫Cookie）保存到本地，在之后的请求中会在header中携带cookie。

## 代码演示

本案例基于Spring Boot 2.7.X + THymeLeaf 实现一个前后端不分离的项目。好处是方便编写代码，不用写前后端两套代码，并用Ajax交换数据。

1. 引入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```



2. 添加ThymeLeaf的配置文件信息：

```properties
spring.thymeleaf.cache=false
spring.thymeleaf.check-template=true
spring.thymeleaf.check-template-location=true
spring.thymeleaf.servlet.content-type=text/html
spring.thymeleaf.enabled=true
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.excluded-view-names=
spring.thymeleaf.mode=HTML
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```



3. 编写Controller文件：



## 细节问题

httponly的cookie无法通过js操作，防止XSS攻击，也无法通过js拦截该cookie。这种cookie通常存放敏感信息。

`Set-Cookie: name=value; HttpOnly`会被设置为httponly
