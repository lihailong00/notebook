# SpringBoot统一结构返回+traceId

[toc]



```java
package com.lee.sec.vo;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class ResponseResult {
    /**
     * 状态码
     * 所有响应均返回200状态码。可以根据个人喜好自定义。
     */
    private Integer code;

    /**
     * 响应是正常还是异常。
     * 正常返回true，异常返回false。
     */
    private Boolean success;

    /**
     * 提示信息，进一步解释响应内容
     */
    private String msg;

    /**
     * 响应数据
     */
    private Object data;

    public static ResponseResult success() {
        return new ResponseResult(200, true, "响应正常", null);
    }

    public static ResponseResult success(Object data) {
        return new ResponseResult(200, true, "响应正常", data);
    }

    public static ResponseResult success(Object data, String msg) {
        return new ResponseResult(200, true, msg, data);
    }

    public static ResponseResult fail(String msg) {
        return new ResponseResult(200, false, msg, null);
    }

    public static ResponseResult fail(String msg, Object data) {
        return new ResponseResult(200, false, msg, data);
    }
}

```

