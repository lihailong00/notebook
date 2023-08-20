# Spring事务

[toc]



## 什么是事务

事务是一组操作，满足四大	原则（ACID）。





## Spring实现事务的两种方法

### 编程式事务

编程式事务使用较少，但有助于理解事务。其中可以使用`TransactionTemplate`或`TransactionManager`管理事务。



>TransactionTemplate

```java
@Resource
private TransactionTemplate transactionTemplate;

public void doTransaction2() {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus status) {
            try {
                // ======业务代码======
                // ......
            } catch (Exception e) {
                e.printStackTrace();
                // 设置事务回滚
                status.setRollbackOnly();
            }
        }
    });
}
```



>TransactionManager

```java
public void doTransaction() {
    // 1.定义默认的事务属性
    DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
    // 2.获取TransactionStatus
    TransactionStatus status = transactionManager.getTransaction(definition);
    try {
        // ======业务代码======
        // ......
        
        // 提交事务
        transactionManager.commit(status);
    } catch (DataAccessException e) {
        e.printStackTrace();
        // 事务回滚
        transactionManager.rollback(status);
    }
}
```





### 声明式事务

声明式事务是最常使用的方式。通常只需要在类或方法上添加`@Transactional`注解即可。

注意，如果在函数内部使用`try-catch`捕获异常，那么`spring-aop`就无法捕获异常信息，因此无法回滚。

举个例子：

```java
@Transactional(rollbackFor = Exception.class)
public void doTransaction() {
    try {
        jdbcTemplate.update("update user set age = ? where username = ?", 8, "丁安琪");
        int x = 1 / 0;
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

上述代码抛出异常后，无法被aop捕获，故无法回滚。



### 事务的隔离性

Spring中，事务的隔离性和MySQL中事务的隔离性是一样的。

1. 读未提交。
2. 读已提交。
3. 可重复读。
4. 串行化。
5. 默认（和数据库底层采用的隔离性一致）。



> 声明式事务中如何配置隔离性

```java
@Transactional(isolation = Isolation.DEFAULT)
public void doTransaction() {
    // 业务代码
}
```



### 事务的传播性

假定调用`doTransaction`函数。



1. 默认情况是`REQUIRED`：如果当前存在事务，则加入当前事务；如果当前不存在事务，则创建一个新事务。

   下面两个案例中，抛出错误后两条数据都会回滚。

> 案例

思考下述案例为什么没有发生死锁？

​    

1. UserService1.java

```java
package com.lee.mytransaction.service;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Component
public class UserService1 {
    @Resource
    private JdbcTemplate jdbcTemplate;

    @Resource
    private UserService2 userService2;

    @Transactional(rollbackFor = Exception.class)
    public void doTransaction() {
        jdbcTemplate.update("update user set age = ? where username = ?", 8, "丁安琪");
        userService2.doTransaction();
        // 地点1
    }
}
```



2. UserService2.java

```java
package com.lee.mytransaction.service;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Component
public class UserService2 {
    @Resource
    private JdbcTemplate jdbcTemplate;

    @Transactional(rollbackFor = Exception.class)
    public void doTransaction() {
        jdbcTemplate.update("update user set age = ? where username = ?", 8, "严安琪");
        // 地点2
    }
}
```



- 在地点1或地点2抛出异常后，两个set语句都需要回滚。






2. `REQUIRES_NEW`：无论当前是否存在事务，都创建一个新的事务。

> 案例

问题：如果`username`没有加索引，为什么会发生死锁？

答案：`username`没有加索引时，set操作会加表锁。事务`userService1`给表加上锁后，事务`userService2`无法操作表，而`userService1`需要等`userService2`执行完毕后才能提交事务。



1. UserService1

```java
package com.lee.mytransaction.service;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Component
public class UserService1 {
    @Resource
    private JdbcTemplate jdbcTemplate;

    @Resource
    private UserService2 userService2;

    @Transactional(rollbackFor = Exception.class)
    public void doTransaction() {
        jdbcTemplate.update("update user set age = ? where username = ?", 8, "丁安琪");
        userService2.doTransaction();
        // 地点1
    }
}
```



2. UserService2

```java
package com.lee.mytransaction.service;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Component
public class UserService2 {
    @Resource
    private JdbcTemplate jdbcTemplate;

    @Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
    public void doTransaction() {
        jdbcTemplate.update("update user set age = ? where username = ?", 8, "严安琪");
        // 地点2
    }
}
```



- 地点1发生异常，不印象`userService2`的正常提交。
- 地点2发生异常，该异常既会传递给`userService2`的`spring-aop`，也会传递给`userService1`的`spring-aop`，故两个事务都会发生回滚。



### 实战

以下案例是基于Spring事务，实现了使用JDBC操作数据库。

1. pom.xml中引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.0.32</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.20</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.3.20</version>
    </dependency>
</dependencies>
```



2. 编写配置类`AppConfig.java`

```java
package com.lee.mytransaction.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.transaction.support.TransactionTemplate;

import javax.sql.DataSource;

@Configuration
@ComponentScan(basePackages = "com.lee.mytransaction")
@EnableTransactionManagement  // 开启事务的注解支持
public class AppConfig {
    // 配置数据源
    @Bean
    public DataSource dataSource() {
        // 配置并返回数据源
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/webproject");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        return dataSource;
    }

    // 配置事务管理器
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    // 配置 TransactionTemplate
    @Bean
    public TransactionTemplate transactionTemplate(PlatformTransactionManager transactionManager) {
        return new TransactionTemplate(transactionManager);
    }

    // 配置 JdbcTemplate
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```



3. 编写声明式事务代码

```java
package com.lee.mytransaction.service;

import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
import org.springframework.transaction.support.TransactionCallbackWithoutResult;
import org.springframework.transaction.support.TransactionTemplate;

import javax.annotation.Resource;
import javax.sql.DataSource;

@Component
public class TransactionExample {
    @Resource
    private DataSource dataSource;
    @Resource
    private TransactionTemplate transactionTemplate;
    @Resource
    private JdbcTemplate jdbcTemplate;
    @Resource
    private PlatformTransactionManager transactionManager;

    // 方法1：使用transactionManager
    public void doTransaction() {
        // 1.定义默认的事务属性
        DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
        // 2.获取TransactionStatus
        TransactionStatus status = transactionManager.getTransaction(definition);
        try {
            jdbcTemplate.update("update user set age = ? where username = ?", 8, "丁安琪");
            transactionManager.commit(status);
        } catch (DataAccessException e) {
            e.printStackTrace();
            transactionManager.rollback(status);
        }
    }

    // 方法2：使用transactionTemplate
    public void doTransaction2() {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                try {
                    jdbcTemplate.update("update user set age = ? where username = ?", 8, "丁安琪");
                    int x = 1 / 0;
                } catch (Exception e) {
                    // 设置事务回滚
                    status.setRollbackOnly();
                    e.printStackTrace();
                }
            }
        });
    }
}
```



4. 或者编写注解式事务

```java
package com.lee.mytransaction.service;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Component
public class TransactionWithAnnotationExample {
    @Resource
    private JdbcTemplate jdbcTemplate;

    @Transactional
    public void doTransaction() {
        jdbcTemplate.update("update user set age = ? where username = ?", 8, "丁安琪");
        int x = 1 / 0;
    }
}
```



5. 主类

```java
package com.lee.mytransaction;

import com.lee.mytransaction.config.AppConfig;
import com.lee.mytransaction.service.TransactionExample;
import com.lee.mytransaction.service.TransactionWithAnnotationExample;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;


public class Main {
    public static void main(String[] args) {
        //  AppConfig 类将作为配置类用于构建 Spring 的应用上下文
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        // 编程式事务
        TransactionExample transactionExample = context.getBean(TransactionExample.class);
        transactionExample.doTransaction();

        // 声明式事务
        TransactionWithAnnotationExample transactionWithAnnotationExample = context.getBean(TransactionWithAnnotationExample.class);
        transactionWithAnnotationExample.doTransaction();
    }
}
```



