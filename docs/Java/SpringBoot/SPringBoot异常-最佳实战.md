# SpringBoot异常-最佳实战



## 准则

1. **不捕获异常就是最好的使用异常的方法**！
2. 尽可能晚的捕获异常，最好在最上层捕获并统一处理这些异常。为避免重复输出异常日志，尽可能在最上层输出异常日志，通常在切入controller的切面中（或@ControllerAdvice+@Exception）捕获异常，为什么我们需要将异常在最上层捕获？我认为有几点原因：
   1. 最上层是最后一步处理异常的时机。统一在最上层处理异常很方便。
   2. 我们通常会将异常信息和普通日志信息输出到不同文件。
3. 抛出具体的异常，而不是Exception。建议自定义两种异常``BizException``和``SysException``，分别记录业务异常和系统异常。
4. 捕获异常后使用语言描述异常。

```Java
log.error("反序列化时出现异常: {}", e.getMessage(), e)
```

5. 不要同时记录和抛出异常（一般建议最上层打日志）。

```Java
// 错误做法：
public String upload(MultipartFile file) throws BizException {
    String originalFilename = file.getOriginalFilename();
    if (StringUtils.isBlank(originalFilename)) {
        log.error("file name can't be null!");
        throw new BizException("file name can't be null!", e);
    }
}
```

6. 优先捕获具体的异常。（通常在controller的切面中捕获异常）

```Java
try {
    work();
} catch (BizException e) {
    // 处理
} catch (Exception) {
    // 处理
}
```

7. 自定义异常不要丢弃原有异常，应该将原始异常传入自定义异常。这种情况通常发生在我们用自己的异常类替换底层包抛出的异常。而我们自己抛出的异常一般直接往上抛。

```Java
throw new SysException("system exception occur", e)
```

8. 不要忽略Throwable异常。



## 自定义异常类

1. BizException

```Java
package com.lhl.minio_demo.exception;

public class BizException extends Exception {
    /**
     * 错误码
     */
    private String code;

    /**
     * 构造一个没有错误信息的 <code>SystemException</code>
     */
    public BizException() {
        super();
    }

    /**
     * 使用指定的 Throwable 和 Throwable.toString() 作为异常信息来构造 SystemException
     *
     * @param cause 错误原因， 通过 Throwable.getCause() 方法可以获取传入的 cause信息
     */
    public BizException(Throwable cause) {
        super(cause);
    }

    /**
     * 使用错误信息 message 构造 SystemException
     *
     * @param message 错误信息
     */
    public BizException(String message) {
        super(message);
    }

    /**
     * 使用错误码和错误信息构造 SystemException
     *
     * @param code    错误码
     * @param message 错误信息
     */
    public BizException(String code, String message) {
        super(message);
        this.code = code;
    }

    /**
     * 使用错误信息和 Throwable 构造 SystemException
     *
     * @param message 错误信息
     * @param cause   错误原因
     */
    public BizException(String message, Throwable cause) {
        super(message, cause);
    }

    /**
     * @param code    错误码
     * @param message 错误信息
     * @param cause   错误原因
     */
    public BizException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    /**
     * 获取错误码
     *
     * @return 错误码
     */
    public String getCode() {
        return code;
    }
}
```

1. SysException

```Java
package com.lhl.minio_demo.exception;

public class SysException extends Exception {
    /**
     * 错误码
     */
    private String code;

    /**
     * 构造一个没有错误信息的 <code>SystemException</code>
     */
    public SysException() {
        super();
    }

    /**
     * 使用指定的 Throwable 和 Throwable.toString() 作为异常信息来构造 SystemException
     *
     * @param cause 错误原因， 通过 Throwable.getCause() 方法可以获取传入的 cause信息
     */
    public SysException(Throwable cause) {
        super(cause);
    }

    /**
     * 使用错误信息 message 构造 SystemException
     *
     * @param message 错误信息
     */
    public SysException(String message) {
        super(message);
    }

    /**
     * 使用错误码和错误信息构造 SystemException
     *
     * @param code    错误码
     * @param message 错误信息
     */
    public SysException(String code, String message) {
        super(message);
        this.code = code;
    }

    /**
     * 使用错误信息和 Throwable 构造 SystemException
     *
     * @param message 错误信息
     * @param cause   错误原因
     */
    public SysException(String message, Throwable cause) {
        super(message, cause);
    }

    /**
     * @param code    错误码
     * @param message 错误信息
     * @param cause   错误原因
     */
    public SysException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    /**
     * 获取错误码
     *
     * @return 错误码
     */
    public String getCode() {
        return code;
    }
}
```



## 实战案例

这里结合`@RestControllerAdvice`捕获并处理异常。

```Java
package com.lhl.minio_demo.handler;

import cn.hutool.json.JSONUtil;
import com.lhl.minio_demo.exception.BizException;
import com.lhl.minio_demo.exception.SysException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    @ExceptionHandler(value = BizException.class)
    public String bizException(BizException e) {  // 返回值通常是对象，可以自定义ResponseResult
        Map<String, String> userInfo = new HashMap<>();
        userInfo.put("username", "lhl");
        // 先打印文字错误提示，再将e传入error函数，打印错误堆栈。
        log.error("业务异常，提示信息: {}\n当前用户信息: {}\n异常堆栈: ", e.getMessage(), JSONUtil.toJsonStr(userInfo), e);
        return "业务异常";
    }

    @ExceptionHandler(value = SysException.class)
    public String sysException(SysException e) {
        log.error("系统异常，提示信息: {}", e.getMessage(), e);
        return "系统异常";
    }

    @ExceptionHandler(value = Throwable.class)
    public String throwable(Throwable e) {
        log.error("未知异常，提示信息: {}", e.getMessage(), e);
        return "未知异常";
    }
}
```
