# SpringBoot JWT

[toc]

## 整合方法

引入依赖：

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.2.2</version>
</dependency>
```



## 自定义JWTUtil类以及测试案例

```java
package com.lee.sec.util;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTCreator;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.DecodedJWT;

import java.nio.charset.StandardCharsets;
import java.util.Base64;
import java.util.Calendar;
import java.util.HashMap;
import java.util.Map;

public class JWTUtil {
    // JWT私钥，自己妥善保管
    private static String secretKey = "jfioewsjfewoia3892udksjhfjo";

    /**
     * 生成token
     * @param data token中封装的数据
     * @param validTime token有效时间
     * @return JWT生成的token字符串
     */
    public static String getToken(Map<String, String> data, int validTime) {
        Calendar instance = Calendar.getInstance();
        // 设置过期时间
        instance.add(Calendar.SECOND, validTime);

        // 创建jwt builder
        JWTCreator.Builder builder = JWT.create();

        // payload
        data.forEach(builder::withClaim);

        // 返回token
        String token = builder.withExpiresAt(instance.getTime()).sign(Algorithm.HMAC256(secretKey));

        return token;
    }

    /**
     * 验证token合法性
     * @param token 前端传入的待检验的token
     */
    public static void checkToken(String token) {
        JWT.require(Algorithm.HMAC256(secretKey)).build().verify(token);
    }

    /**
     * 获取token信息
     * @param token
     * @return
     */
    public static DecodedJWT getTokenInfo(String token) {
        return JWT.require(Algorithm.HMAC256(secretKey)).build().verify(token);
    }

    /**
     * 测试案例
     * @param args
     */
    public static void main(String[] args) {
        // =============================生成JWTToken==============================

        // 设置jwt的载荷
        // 注意value的类型是string
        Map<String, String> data = new HashMap<>();
        data.put("id", "1");
        data.put("name", "lhl");
        // 生成JWT令牌，有效时间单位：秒
        String token = JWTUtil.getToken(data, 7 * 24 * 3600);

        // ==============================验证JWTToken=============================

        // 返回给前端的响应
        ResponseResult responseResult = null;
        try {
            JWTUtil.checkToken(token);
            responseResult = ResponseResult.success();
        } catch (SignatureVerificationException e) {
            e.printStackTrace();
            responseResult = ResponseResult.fail("无效签名！");
        } catch (TokenExpiredException e) {
            e.printStackTrace();
            responseResult = ResponseResult.fail("token过期");
        } catch (AlgorithmMismatchException e) {
            e.printStackTrace();
            responseResult = ResponseResult.fail("token算法不一致");
        } catch (Exception e) {
             e.printStackTrace();
            responseResult = ResponseResult.fail("token错误");
        }
        
        // 解析JWT令牌中的信息
        DecodedJWT tokenInfo = JWTUtil.getTokenInfo(token);

        System.out.println("jwt令牌内容：" + tokenInfo);
        System.out.println("载荷内容：" + new String(Base64.getDecoder().decode(tokenInfo.getPayload()), StandardCharsets.UTF_8));
        // 获取数据时，先调用asString转成字符串，再进行其他操作！
        System.out.println("id=" + tokenInfo.getClaim("id").asString());
        System.out.println("过期时间：" + tokenInfo.getExpiresAt());
        
        // 返回responseResult给前端
        // return responseResult;
    }
}
```







## JWT结合拦截器

```java
```

