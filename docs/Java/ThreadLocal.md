# ThreadLocal

[toc]



## 是什么

核心：`ThreadLocal`是一个类，我们通常在**`ThreadLocal`对象中存放只属于某个线程的全局变量**。

不严谨的说，使用`ThreadLocal`对象可以跨方法，跨类共享变量。

想要获取threadlocal中的值时，只需要new一个threadlocal对象就行（哪个地方new都可以）

工作原理：每个线程对象中，都有一个`ThreadLocalMap`对象，只需要将值存入`ThreadLocalMap`对象中即可。

在说`ThreadLocal`之前，我们先了解什么是**线程封闭**。

线程封闭，也即将对象放在线程里，**让对象只属于某个线程**，这样就能解决并发问题。

实现线程封闭有以下两种方式：

1. 将线程对象放入栈中，修改局部变量不会产生线程安全问题。

   ```java
   public void test() {
       StringBuilder sb = new StringBuilder();
       sb.append("hi");
   }
   ```

2. 将对象放在`ThreadLocal`中。



## 使用方法

复制下面的代码，注意UserDTO是我自己的类，根据自己业务选择合适的类。

```java
public class UserHolder {
    private static final ThreadLocal<UserDTO> tl = new ThreadLocal<>();
    public static void saveUser(UserDTO user){
        tl.set(user);
    }
    public static UserDTO getUser(){
        return tl.get();
    }
    public static void removeUser(){
        tl.remove();
    }
}
```



这样使用：

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    // 判断是否需要拦截（ThreadLocal中是否有用户）
    if (UserHolder.getUser() == null) {
        response.setStatus(401);  // 没有，需要拦截，设置状态码
        return false;  // 拦截
    }
    // 有用户，则放行
    return true;
}
```

