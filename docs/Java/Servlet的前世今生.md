# Servlet和JSP的前世今生

[toc]



## 什么是Servlet

`Servlet`实际上就是`java.servlet`包里的一个`interface`。具体代码如下：

```java
package javax.servlet;

import java.io.IOException;

public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```

`service`方法是最重要的，它的作用是把`tomcat(servlet容器)`传入的请求进行处理，并返回结果。

`tomcat`封装客户端`Socket`传入的请求，将其变为`ServletRequest`类型。



补充：web服务器是一种软件，可以将本地资源映射为`URL`暴露给外界。



总结：`servlet`就是一个接口，它为Java程序提供了一个统一的web应用规范。





## JSP

我们都知道，浏览器访问web服务器后，通常会得到一个`html`文件，文件内容大致如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>hello, world!</h1>
</body>
</html>
```

那么服务器是如何生成上述html文本呢？

最早期实际上是美工负责写HTML，然后Java`程序员将写好的HTML代码嵌入Java代码中，如下所示：

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    response.setContentType("text/html");
    PrintWriter out = response.getWriter();
    out.println("<!DOCTYPE html>");
    out.println("<html lang="en">");
    out.println("<head>");
    out.println("    <meta charset="UTF-8">");
    out.println("    <meta http-equiv="X-UA-Compatible" content="IE=edge">");
    out.println("    <meta name="viewport" content="width=device-width, initial-scale=1.0">");
    out.println("    <title>Document</title>");
    out.println("</head>");
    out.println("<body>");
    out.println("    <h1>hello, world!</h1>");
    out.println("</body>");
    out.println("</html>");
}
```

那么这就有两个问题：

- 一个HTML页面动辄几千行代码，Java程序员一行一行粘贴进去，可能会被累死~
- 页面是静态的。也就是说不同的人访问该服务器，得到的HTML页面是相同的。而当今绝大多数网站是动态的，比如我访问的B站内容几乎全是宅舞区视频，而小明访问的B站全是学习视频。B站根据不同的人物推荐不同的内容。



此时JSP诞生了！

JSP（Java Server Page），也即“运行在服务器端的页面”。我们可以在JSP文件中写HTML代码，还可以在JSP文件中写Java代码。服务器会将JSP页面动态渲染为HTML文件，然后返回给客户端。

下述案例是将Java集合中的一些姓名动态渲染到页面中的一个例子。

```java
<%@ page contentType="text/html;charset=UTF-8" %>
<!DOCTYPE html>
<html>
<head>
    <title>姓名列表</title>
</head>
<body>
    <h1>姓名列表</h1>
    <ul>
        <% 
        List<String> nameList = new ArrayList();
        nameList.addName("John");
        nameList.addName("Jane");
        nameList.addName("Alice");
        for (String name : nameList) {
            out.println("<li>" + name + "</li>");
        }
        %>
    </ul>
</body>
</html>
```



那么我们想想，JSP是如何被服务器渲染为HTML文件的呢？具体过程如下：

1. web服务器收到一个访问`jsp`页面的请求。
2. JSP引擎将JSP文件编译成一个`Servlet`，该`Servlet`类会实现`javax.servlet.Servlet`接口，并包含JSP文件中的静态内容和动态代码。
3. 编译后的Servlet类会被加载到内存中，并实例化一个Servlet对象。
4. 调用Servlet对象的service方法，处理客户端的请求。
5. 在 `service()` 方法中，Servlet 对象可以执行 Java 代码、访问数据库、调用其他 Java 类等。它可以生成动态内容并与静态内容结合。
6. Servlet 对象生成的动态内容最终会被发送回客户端作为 HTTP 响应。
7. 客户端接收到响应，并将其渲染为可见的网页或其他类型的内容。



通过上述过程，我们可以知道：为了不让Java程序员一行行复制HTML代码到Servlet里，SUN公司干脆把Java代码和HTML代码杂糅在一起，美其名曰`JSP`。



总结，**JSP = HTML + Java片段**，**JSP就是一个Servlet**！



## servlet对象的生命周期

一个`WebServlet`注解对应一个`Servlet`类，一个`Servlet`类可能会创建多个`servlet`对象。servlet对象负责处理并响应用户请求等功能。



## servlet重要对象

### ServletContext

一个Web应用中只有一个`ServletContext`对象，用于存放全局变量。
