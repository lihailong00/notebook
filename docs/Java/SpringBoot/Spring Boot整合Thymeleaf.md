# Spring Boot整合ThymeLeaf

[toc]

## 写在前面

ThymeLeaf用于前后端耦合的项目，通常情况下我都不会使用。只有在需要创建一个简单的前后端demo时用到ThymeLeaf。



## 代码

引入依赖：pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```



编写关于ThymeLeaf的配置文件：application.properties

```properties
# 生产环境false，线上环境true
# 设置为true，提高了并发能力，但是变更后不能及时更新
spring.thymeleaf.cache=false
```



配置controller：

```java
package com.lee.helloworld.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("")
    public String getInfo(Model model) {
        // 向model中存入动态参数，用于界面的动态渲染
        model.addAttribute("msg", "你好呀！");
        // 返回 hello.html文件（html后缀在配置文件中已指定）
        return "hello";
    }
}
```



编写html文件：`/src/main/resources/templates/hello.html`

```html
<!DOCTYPE html>
<!--注意html标签的属性-->
<html lang="ch" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Hello World!</h1>
    <h2>获取动态数据:<span th:text="${msg}"></span></h2>
</body>
</html>
```

`

## 坑

`pom.xml`中不要配置`resouces`！不然会500错误！
