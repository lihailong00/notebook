# Mybatis-Plus操作MySQL

[toc]



## 前置知识

建议先看完我写的《MybatisX插件》那篇文章。Mybatis-Plus只能做单表查询，如果要多表查询，需要自己在`mapper.xml`文件中写SQL语句。



## springboot集成mybatis-plus



### 确保开启MySQL数据库

检查是否开启mysql数据库。



### 创建Spring Boot项目

使用idea自带的spring initializer即可，也可以自己通过maven配置项目。



### 添加依赖

建议在pom文件中添加以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
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
    
    <!--mysql驱动-->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.0.32</version>
    </dependency>

    <!--mybatis-plus-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.3.1</version>
    </dependency>

    <!--lombok-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```



## mapper.xml位置

假定`mapper`文件在`src/main/java/com/lee/pro/mapper`，则`mapper.xml`文件在`src/main/resources/com/lee/pro/mapper`。



### 设置配置文件

```properties
spring.datasource.username=xxxx
spring.datasource.password=xxxx
spring.datasource.url=jdbc:mysql://localhost:3306/xxxxxx?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# 打印SQL语句
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```



### 测试是否连接上数据库

```java
@SpringBootTest
class MysqlTestApplicationTests {
    @Resource
    private DataSource dataSource;
    @Test
    void contextLoads() throws SQLException {
        System.out.println(dataSource.getConnection());
    }
}
```



### 使用MybatisX插件生成代码

参考我写的《MybatisX插件》那篇文章。此时会生成一个`实体类Java代码`、`service和serviceImpl代码`、`mapper接口代码`、`mapper.xml文件`。注意`mapper.xml`文件要放在`resources`目录下并且其所属包名和Java代码中的`mapper接口代码`的所属包名相同。

例如：`/src/main/java/com/lee/mysqltest/mapper/UserMapper`对应`/src/main/resources/com/lee/mysqltest/mapper/UserMapper.xml`。



## 添加MapperScan注解

在启动类上添加MapperScan注解。

`@MapperScan("com.lee.pro.mapper")`

也可以在每个`mapper`的接口文件中加入注解`@Mapper`，不过这样比较麻烦。



## CRUD接口

参考[官方文档](https://baomidou.com/pages/49cc81/#service-crud-%E6%8E%A5%E5%8F%A3)。

我们可以通过Mybatis-Plus提供的**service层接口**或**mapper层接口**操作数据库。个人习惯对于简单的查询用service提供的接口；对于多表查询使用子查询或在mapper中写SQL语句，不使用mapper提供的查询方法。



## CRUD案例

查找：

```java
package com.lee.mybatisplugin;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.lee.mybatisplugin.pojo.User;
import com.lee.mybatisplugin.service.UserService;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;
import java.util.List;

@SpringBootTest
class MybatisPluginApplicationTests {
    @Resource
    private UserService userService;

    @Test
    void test() {
        // from user
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        // SELECT u_id, name
        queryWrapper.select(User::getUId, User::getName);
        // WHERE name = Mark
        queryWrapper.eq(User::getName, "Mark");
        // WHERE age > 8
        queryWrapper.gt(User::getAge, 8);
        // ORDER BY create_time desc
        queryWrapper.orderByDesc(User::getCreateTime);
        // 执行 多个查询
        List<User> users = userService.list(queryWrapper);
        // 执行 单个查询
        User user = userService.getOne(queryWrapper);

        System.out.println(users);
    }
}
```





## 小知识点

1. querywrapper和lambdaquerywrapper的区别？
