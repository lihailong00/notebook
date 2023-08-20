# springboot基础案例

[toc]

## 前言

创建一个完整的springboot项目，我们需要按照以下方法来做。

1. 使用idea正确创建一个空的springboot项目。
2. 通过maven工具，在pom.xml文件中导入必要的依赖。
3. 创建controller、service、mapper三层文件夹。service要分清接口和类实现。
4. 合理配置application.properties文件，也考虑采用yaml格式的配置文件。可参考我的另一篇专门关于配置文件的博客。
5. 采用mybatis+MySQL的方式连接数据库。可参考我的另一篇专门写mybatis的博客。



以下案例提供了一个简单的springboot项目的创建方法。



## 前置工具

1. idea（专业版）
2. jdk 1.8
3. maven

备注：可使用idea自带的jdk和maven。



## 具体步骤

### 1. 使用idea初始化空的springboot项目。

![QQ截图20221221160131](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/QQ%E6%88%AA%E5%9B%BE20221221160131.png)



![QQ截图20221221172610](C:\Users\晓龙coding\Pictures\QQ截图20221221172610.png)



### 2. 导入必要依赖

部分依赖是springboot项目自带的，部分是需要自己添加的。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!--web服务模块-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--测试模块-->
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-launcher</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```





### 3. 创建三层文件

按照如下规范创建controller、service和mapper层以及一些必备的文件。

注意，一定要分清哪些是类，哪些是接口！

![QQ截图20221221202554](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/QQ%E6%88%AA%E5%9B%BE20221221202554.png)



controller层的内容：

```java
package com.lee.sp1.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
@ResponseBody
@RequestMapping("/login")
public class LoginController {
    @GetMapping("/port1")
    public String port1() {
        return "this is port 1";
    }
    @PostMapping("/port2")
    public String port2(@RequestBody String data) {
        return data;
    }
}
```

可以运行浏览器访问web服务是否正常。



以上只是简单的案例，以上案例没有涉及到mybatis连接MySQL等操作。

