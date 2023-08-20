# springboot访问静态资源

[toc]



## 静态文件存放位置

`/src/main/resources/public`或

`/src/main/resources/static`。

个人习惯将对外暴露的资源放在public中。

不建议自定义静态文件的存放位置。



## 访问静态资源

假定我们将图片`a.png`放在上述目录下，我们仅需要访问`http://localhost:8080/a.png`即可。

如果我们想对url自定义前缀，只需要在`application.properties`中填写如下配置即可。

```properties
spring.mvc.static-path-pattern=/images/**
```

此时访问：`http://localhost:8080/images/a.png`

注：有时候需要清空缓存才能加载正确。



## 完美解决SpringBoot上传图片之后，需要重服务才能访问

出现原因：

说法一：这样导致的原理是服务器的保护措施导致的，服务器不能对外部暴露真实的资源路径，需要配置虚拟路径映射访问。

说法二：SpringBoot 在启动的时候会执行maven命令把本地项目打包成jar包或war包，其中就包含了项目中的静态资源文件。启动项目后上传的图片并未传入到启动的包中，而是在本地的项目中，所以访问不到。项目重启后又会将本地项目打成新的包，此时包含上一次上传的的图片，于是就会在页面上显示图片。**个人更偏向于这种说法。**



解决办法：添加如下配置类即可。

```java
package com.lee.uploadfile.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class FileUploadConfig implements WebMvcConfigurer {
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        // 获取文件的真实路径
        // 注意：末尾的\\必须要写！
        String path = System.getProperty("user.dir")+"\\src\\main\\resources\\public\\";
        // 添加映射
        // 前者是url的前缀，后者是文件实际路径（注意加上file:）
        registry.addResourceHandler("/images/**").addResourceLocations("file:" + path);
   }
}
```

