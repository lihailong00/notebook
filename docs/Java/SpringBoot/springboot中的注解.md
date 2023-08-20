# SpringBoot中的注解

[toc]

## Controller层注解



### @RequestMapping

作用：将请求映射到类定义或方法定义处。

**末尾不要写斜线！**

前面可写斜线，也可不写斜线。**建议前面写上斜线。**

```java
@Controller
public class LoginController {
    // 访问login路径
    @RequestMapping("/login")
    public String func() {
        // 跳转到src/resources/static目录
        return "login.html";
    }
}
```



`spring.mvc.view.prefix`：给`@RequestMapping`下面的函数的返回值前面拼接一串字符。

`spring.mvc.view.suffix`：给`@RequestMapping`下面的函数的返回值后面拼接一串字符。



### @GetMapping

 `@GetMapping` = `@RequestMapping(method = RequestMethod.GET)`

### @PostMapping

 `@PostMapping` = `@RequestMapping(method = RequestMethod.POST)`



### @MapperScan

写在SpringBoot的启动程序上面。例如：`@MapperScan("com.lee.project.mapper")`



### @Repository

`@Repository`和`@Controller`、`@Service`、`@Component`的作用差不多，都是把对象交给spring管理。@Repository用在持久层的**接口**上，这个注解是将接口的一个实现类交给spring管理。



### @Controller

`@Controller`类中的方法可以直接通过返回String跳转到jsp、ftl、html等模版页面。

```java
@Controller
public class ViewController {
    @RequestMapping("/login")
    public String login() {
        // 跳转到static目录下的loginPage.html文件
        return "loginPage.html";
    }
}
```

### @ResponseBody

```java
@Controller
@ResponseBody
public class ViewController {
    @RequestMapping("/login")
    public String login() {
        // 直接在当前界面打印json字符串"loginPage.html"
        return "loginPage.html";
    }
}
```

注意：`@ResponseBody`也可以放在函数上面，仅让部分函数可以直接返回json数据。



### @RestController

`@RestController` = `@Controller` + `@ResponseBody`



### @RequestBody

 @RequestBody用来接收前端传递给后端请求。支持Post请求，不支持Get请求。在后端的同一个接收方法里，@RequestBody与@RequestParam()可以同时使用，**@RequestBody最多只能有一个**，而@RequestParam()可以有多个。

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @PostMapping
    public String getInfo(@RequestBody String info) {
        return info;
    }
}
```

你以为`@RequestBody`只能接受String类型的参数？大错特错！**它可以将接收到的json数据自动转换成Java的实体类！**看个案例：

```java
// pojo 中
@Data
public class SysUser {
    private String username;
    private String password;
}

@RestController
@RequestMapping("/user")
public class UserController {
    @PostMapping("/show")
    public String show(@RequestBody SysUser sysUser) {
        return JSON.toJSONString(sysUser);
    }
}
```

发送post请求，消息主体中的json数据是`{ username: lhl, ratings: 3600 }`。尽管和SysUser中的数据不是完全匹配，但是仍然能匹配部分数据，最终返回的结果是向请求方返回的结果是：`{"username":"lhl"}`。



### @RequestParam

接收前端传递给后端的请求。既支持Post请求，也支持Get请求。@RequestParam()可以有多个。

```java
// 访问http://localhost:8080/param?id=11&type=apple
// 在界面上输出 "id=11 type=apple"
@RestController
public class ViewController {
    @RequestMapping("/param")
    public String Params(@RequestParam(value = "id") int id, @RequestParam(value = "type")String type) {
        String info = "id=" + id + " type=" + type;
        return info;
    }
}
```



### @PathVariable





### @CrossOrigin



## Service层注解

### @Service

注册bean。写在Service实现类上面（不是Service接口上面）。本质上是`@Component`。



### @Component

注册bean。



### @Bean

注册bean。写在函数上面，让IOC容器接管函数生成的对象。

`@Bean`的优点是可以让IOC容器接管第三方库中的对象。

使用`@Bean`就得使用`@Configuration`。`@Bean`写在方法上，`@Configuration`写在类上。



使用案例：

```java
@Configuration
class Person {
    @Bean
    public Person getPerson() {
        return new Person();
    }
    void talk() {
        System.out.println("这个人正在唠叨。");
    }
}

@SpringBootTest
class TestClass {
    @Autowired
    @Qualifier("getPerson")  // @Bean下的函数名
    private Person person;
    @Test
    void test01() {
        person.talk();
    }
}
```



### @Configuration

使用`@Bean`就得使用`@Configuration`。`@Bean`写在方法上，`@Configuration`写在类上。



### @Autowired

获取bean。一般获取Service接口的对象（而不是获取ServiceImpl实现类的对象）。

如果接口只对应一个实现，`@Autowired`和`@Resource`是一样的。

如果接口对应多个实现，`@Autowired` + `@Qualifier`("xxx") = `@Resource`("xxx")。

注意：实现类xxx一般是大驼峰写法，而作为注解的参数是小驼峰写法。

### @Resource

获取bean。见`Autowired`的使用方法。

一个关于`@Resource`和`Autowired`的区别的案例：

```java
interface PersonService { 
    void work();
}

@Service
class StudentServiceImpl implements PersonService {
    @Override
    public void work() {
        System.out.println("学生在学习");
    }
}

@Service
class TeacherServiceImpl implements PersonService {
    @Override
    public void work() {
        System.out.println("老师在教书");
    }
}

@SpringBootTest
class TestClass {
    @Autowired
    @Qualifier("teacherServiceImpl")
    private PersonService personService1;

    @Test
    void test01() {
        personService1.work();
    }

    @Resource(name = "studentServiceImpl")
    private PersonService personService2;
    @Test
    void test02() {
        personService2.work();
    }
}
```





## Mapper层注解

### @Mapper

写在每个Mapper接口上面，或者在spring boot启动处一次性使用MapperScan(mapper文件集合所在位置)。



## 其他注解

### @Transactional

目前据我所知，用作事务回滚。



### @Value

获取`application.properties`中的值。

案例：

> application.properties

```properties
weixin.appid=wxxxxx
weixin.secret=aaabbbddd
```

> 测试代码

使用`${}`解析application.properties中的值。

```java
import org.springframework.beans.factory.annotation.Value;
@SpringBootTest
class Test {
    @Value("${weixin.appid}")
    private String appid;
    @Value("${weixin.secret}")
    private String secret;
    @Test
    void test() {
        System.out.println(appid + "," + secret);
    }
}
```





## 其他知识

1. 任何函数前必须加上`@RequestMapping`、`@GetMapping`或`@PostMapping`才能处理url请求。
