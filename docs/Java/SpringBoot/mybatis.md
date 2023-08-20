# Mybatis 笔记

[toc]

## 写在前面

个人习惯使用Mybatis，因为直接操作SQL语句会很酷（虽然有一定的门槛）。

Mybatis在Spring Boot项目中的使用方法和官方文章中的有一些区别，因此我才写下这篇关于文章Mybatis在Spring Boot中的使用方法。

建议安装`MybatisX`插件。



## 项目结构

```
│  pom.xml
│
├─src
│  ├─main
│  │  ├─java
│  │  │  └─com
│  │  │      └─lee
│  │  │          └─mybatis
│  │  │              │  MybatisApplication.java
│  │  │              │
│  │  │              ├─mapper
│  │  │              │      UserMapper.java
│  │  │              │
│  │  │              └─pojo
│  │  │                      User.java
│  │  │
│  │  └─resources
│  │      │  application.properties
│  │      │
│  │      └─mybatis
│  │          └─mapper
│  │                  UserMapper.xml
│  │
│  └─test
│      └─java
│          └─com
│              └─lee
│                  └─mybatis
│                          MybatisApplicationTests.java
```



## 代码

#### pom.xml

```xml
<dependencies>
    <!--springboot的基本配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--mysql配置-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <!--mybatis相关依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.10</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.2</version>
    </dependency>

    <!--其他工具-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-launcher</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```



#### application.properties

1. mysql注意选择自己的端口号(通常是3306)和数据库名

2. `mybatis.mapper-locations`属性定义`Mapper.xml`文件所在位置，一定要写！
3. classpath指的是src.main.java和src.main.resources路径以及第三方jar包的根路径



```properties
# database
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.url=jdbc:mysql://localhost:3306/webproject?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=utf-8
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# mybatis
# 定位Mapper.xml文件所在路径
mybatis.mapper-locations=classpath:mapper/*.xml
```



#### test案例

在`/src/test/jafa/....../`中随便创建一个test案例。

```java
@Test
void test01() {
    List<User> users = userMapper.ListUser();
    if (users == null) {
        System.out.println("数据表为空");
    } else {
        for (User user : users) {
            System.out.println(user);
        }
    }
}
```



#### MybatisApplication.java

启动类上面一定要添加`@MapperScan("com.lee.mybatis.mapper")`，这样才能扫描到Mapper接口文件，否则需要在每个Mapper接口上面添加`@Mapper`。

```java
package com.lee.mybatis;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("com.lee.mybatis.mapper")
public class MybatisApplication {
    public static void main(String[] args) {
        SpringApplication.run(MybatisApplication.class, args);
    }
}
```



#### User.java

User实体类一定要和数据表User严格对应！

```java
package com.lee.mybatis.pojo;

import lombok.Data;

@Data
public class User {
    private Long id;
    private String name;
    private int age;
    private String email;
}
```



#### UserMapper.java

```java
package com.lee.mybatis.mapper;

import com.lee.mybatis.pojo.User;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface UserMapper {
    // ListUser是我们自定义的查询方式，在UserMapper.xml中有具体的实现方式。
    List<User> ListUser();
}
```



#### UserMapper.xml

namespace指java文件中对应的Mapper接口文件（不妨叫它doc）。

id属性指doc中需要对应的函数。

resultType指函数的返回类型。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lee.mybatis.mapper.UserMapper">
    <select id="ListUser" resultType="User">
        select * from user
    </select>
</mapper>
```



## MyBatisPlus通过Mapper.xml进行SQL操作

具体操作和Mybatis几乎一样。不同点如下：

1. 引入依赖`mybatis-plus-boot-starter`。

2. `application.properties`文件中的配置：

   ```xml
   mybatis-plus.mapper-locations=classpath:mybatis-plus/mapper/*.xml
   mybatis.type-aliases-package=com.lee.mybatisplus.pojo
   ```



## 使用Mybatis查询部分字段 + 分页查询

假定有一张user表，包含以下属性：

```
id
username
password
email
```

如果我们只想取出`id`和`email`字段，该怎么办呢？

方法一：创建一个新的类，只包含`id`和`email`。（不可取！）

方法二：使用Map存`字段`和`值`。



### 案例

`@MapKey("id")`表示将查询的结果按照id归档。



> UserMapper接口

```java
import org.apache.ibatis.annotations.MapKey;
import org.apache.ibatis.annotations.Param;

import java.util.List;
import java.util.Map;

public interface UserMapper {
    public List<Map<String, Object>> listPartInfo(@Param("start") int start, @Param("len") int len);

    @MapKey("id")
    public Map<String, Object> listPartInfo2(@Param("start") int start, @Param("len") int len);
}
```

> UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lee.weapp.mapper.UserMapper">
    <select id="listPartInfo" resultType="java.util.Map">
        select id, email from user order by id limit #{start}, #{len}
    </select>
    <select id="listPartInfo2" resultType="java.util.Map">
        select id, email from user order by id limit #{start}, #{len}
    </select>
</mapper>
```



查询代码：

```java
@Test
void contextLoads() throws SQLException {
    List<Map<String, Object>> lists = userMapper.listPartInfo(4, 8);
    for (Map<String, Object> mp : lists) {
        System.out.println(mp);
    }

    Map<String, Object> lists2 = userMapper.listPartInfo2(4, 7);
    for (Map.Entry<String, Object> entry : lists2.entrySet()) {
        System.out.println(entry);
    }
}
```



这里给了两种查询方式。**建议使用第二种。**





## Mybatis最简单的使用方法

1. 使用select注解，这样就可以不使用xml配置，也不用使用`@Param`注解了。
2. 通过Java的Map向MySQL传递参数。直接上代码（关于登录的mapper查询）。

**mapper层：**

> LoginController.java

```java
public interface LoginMapper {
    // #{username}会被自动解析为user中key为username所对应的value
    @Select("select * from nltx_user where username = #{username} and password = #{password}")
    Map<String, Object> login(Map<String, Object> user);
}
```

> 测试代码

```java
@Test
void contextLoads() throws SQLException {
    Map<String, Object> user = new HashMap<>();
    user.put("username", "Yin Jialun");
    user.put("password", "oRfH0taAHt");
    Map<String, Object> ansUser = loginMapper.login(user);
    System.out.println(ansUser);
}
```

