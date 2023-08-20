# springboot项目部署与停止

[toc]



## 部署springboot项目

1. 使用Maven管理项目，在`pom.xml`文件中引入插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
    <!--自定义生成的包名-->
    <finalName>xnxy</finalName>
</build>
```

2. 在IDEA界面的右侧，点击`Maven`选项，展开`Lifecycle`，依次点击`clean`、`compime`和`package`插件，即可生成相关jar包。

![QQ截图20230205140910](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/QQ%E6%88%AA%E5%9B%BE20230205140910.png)

3. 将生成的jar包上传到服务器的任意位置，执行指令`nohup java -jar 包名.jar`即可成功部署。



补充：

1. 自定义服务端口：服务端口由源代码的`application.properties`文件指定，默认是8080。如果我们想要自定义端口，只需输入指令`nohup java -jar 包名.jar --server.port=18080 `

2. 创建日志文件：输入指令`nohup java -jar 包名.jar & `，即可在部署服务的同时，生成日志文件。默认会将日志文件输出到当前文件夹下的`nohup.out`文件中。



## 停止springboot服务

> 方法一

1. 输入`ps -ef | grep java`，查询与Java相关的进程。
2. 输入`kill -15 进程号`即可杀死该进程。（不要用kill 9，否则会导致部分资源无法释放！）