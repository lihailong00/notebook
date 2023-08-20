# Junit测试

[toc]

## 写在前面

本文介绍了关于springboot项目中如何使用Junit5做简单的测试，并且只涉及到个人常用的知识点，部分知识未做介绍。

**我最常使用到的功能就是测试mapper，service。**

所有的测试代码都应该放在`/src/test`目录内。



## 相关依赖

引入依赖

```xml
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <scope>test</scope>
</dependency>
```



## 测试流程

软件会在`/src/test`目录中创建一个`xxxTests`类，并且该类上面有`@SpringBootTest`注解。

所有的测试都放在这个类中（也可以自己创建一个类，记得加上`@SpringBootTest`注解）。下面给出使用案例，主要用到`@Test`和`Assertions.xx`方法

```java
@SpringBootTest
class MyTests {
    // 注册bean
    @Autowired
    private UserService userService;
    @Test
    void contextLoads() {
        String password = userService.selectPasswordByUsername("lhl");
        // 使用Assertions中的方法
        Assertions.assertEquals("666", password);
    }
}
```

非常简单！



下面是`@BeforeAll`、`@BeforeEach`、`@AfterAll` 、`@AfterEach`、`@RepeatTest(x)`的使用方法，顾名思义，非常简单。

```java
@SpringBootTest
class MyTests {
    @Autowired
    private UserService userService;
    @BeforeAll
    // 须要静态方法
    static void init() {
        System.out.println("只有第一次测试才会执行这行代码");
    }
    @BeforeEach
    // 须要非静态方法
    void initEach() {
        System.out.println("每做一次测试，都会执行这行代码");
    }
    @RepeatedTest(5)
    void contextLoads() {
        String password = userService.selectPasswordByUsername("lhl");
        Assertions.assertEquals("666", password);
    }
    @AfterEach
    void endEach() {
        System.out.println("做完一次测试，会执行这行代码");
    }
    @AfterAll
    static void end() {
        System.out.println("所有执行完毕，执行这行代码");
    }
}
```



