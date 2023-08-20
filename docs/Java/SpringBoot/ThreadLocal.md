# ThreadLocal使用教程

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

