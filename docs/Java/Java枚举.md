# Java枚举

[toc]



## 为什么要使用枚举

增强可读性。

限定取值范围。



## 案例

基本案例：

```java
enum Fruit {
    APPLE,
    BANANA,
    GRAPE
}

public class Main {
    public static void main(String[] args) {
        Fruit apple = Fruit.APPLE;
        Fruit banana = Fruit.BANANA;
        System.out.println(apple);  // 输入 APPLE
        System.out.println(banana);  // 输出 BANANA
        if (apple == Fruit.APPLE) {
            System.out.println("They are the same!");
        }
    }
}
```



常见案例：

```java
enum ErrorCode {
    OK(200, "OK"),
    BAD_REQUEST(400, "Bad Request"),
    UNAUTHORIZED(401, "Unauthorized"),
    FORBIDDEN(403, "Forbidden"),
    NOT_FOUND(404, "Not Found"),
    INTERNAL_SERVER_ERROR(500, "Internal Server Error");

    private final int code;
    private final String message;

    ErrorCode(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}

public class Main {
    public static void main(String[] args) {
        ErrorCode error = ErrorCode.NOT_FOUND;
        System.out.println("错误代码：" + error.getCode());
        System.out.println("错误信息：" + error.getMessage());
    }
}
```



## 基本内容

参考上述的常见案例。

枚举类型中，通常有以下几类成员：

1. 常量成员：`OK`, `BAD_REQUEST`, `UNAUTHORIZED`等。一个常量成员中可以存放多个值。
2. 构造函数：`ErrorCode`函数，决定了一个常量成员可以几种类型的值。
3. 方法：`getCode`, `getMessage`函数。可以获取一个常量成员的某个值。
