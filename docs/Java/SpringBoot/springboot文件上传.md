# springboot文件上传

[toc]

## 图片上传到服务器



### 后端代码

```java
package com.lee.uploadfile.controller;

import org.springframework.boot.system.ApplicationHome;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.UUID;

/**
 * 可以上传任何类型的文件。主要用于上传图片。
 * @author 晓龙coding
 */
@RestController
@RequestMapping("/upload")
public class UploadController {
    /**
     * 上传单个文件到本地目录
     * @param file
     * @return
     */
    @PostMapping("/single-file")
    String uploadPicture(MultipartFile file) {
        // file校验
        if (file.isEmpty()) {
            return "文件上传失败！";
        }

        // 限定文件大小
        // 也可以在配置文件中设定，下面的案例采用配置文件中设定文件大小
        int maxFileSize = 2 * 1024 * 1024;
        if (file.getSize() > maxFileSize) {
            return "文件不能超过2MB";
        }
        
        // 截取文件名，并用新的文件名替换
        String originalFilename = file.getOriginalFilename();
        assert originalFilename != null;
        int startIndex = originalFilename.lastIndexOf('.');
        String ext = originalFilename.substring(startIndex);
        String randomName = UUID.randomUUID().toString().replace("", "");
        String newFilename = randomName + ext;

        // 上传图片
        ApplicationHome applicationHome = new ApplicationHome(this.getClass());
        String pre = applicationHome.getDir().getParentFile().getParentFile().getAbsolutePath() +
                "\\src\\main\\resources\\public\\";
        String path = pre + newFilename;
        try {
            file.transferTo(new File(path));
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        // 将path返回给前端，前端会将path作为图片的url
        // 有时候可能需要返回字典对象，前端才能正常接收
        return path;
    }
}
```



### 前端代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form action="http://localhost:8080/upload/single-file" method="post" enctype="multipart/form-data">
        <!-- name 属性用于对提交到服务器后的表单数据进行标识 -->
        <input type="file" name="file" value="上传图片">
        <input type="submit" value="上传">
    </form>
</body>
</html>
```



## 图片上传到七牛云

[参考文档](https://developer.qiniu.com/kodo/1239/java#server-upload)

### 导入maven依赖

```xml
<dependency>
  <groupId>com.qiniu</groupId>
  <artifactId>qiniu-java-sdk</artifactId>
  <version>[7.7.0, 7.10.99]</version>
</dependency>
```



### 设置配置文件application.properties

```properties
# 单次请求最大上传尺寸
spring.servlet.multipart.max-request-size=5MB
# 最大上传尺寸
spring.servlet.multipart.max-file-size=1MB

qiNiu.accessKey=z76Agk0XbS38XpDqHPHR_SjeEY920war5qYbQxkv
qiNiu.accessSecretKey=qH7d1b2xTtLGj63tWYh4xI_3aykoN2gnTMq3ZAC3
qiNiu.bucketName=longcoding-wxapp
# 七牛云会分配临时域名，也可以绑定自己的域名
qiNiu.url=http://lihailong.vip/
```





### 自定义工具类

```java
package com.lee.uploadfile.util;

import com.alibaba.fastjson2.JSON;
import com.qiniu.http.Response;
import com.qiniu.storage.Configuration;
import com.qiniu.storage.Region;
import com.qiniu.storage.UploadManager;
import com.qiniu.storage.model.DefaultPutRet;
import com.qiniu.util.Auth;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

/**
 * @author 晓龙coding
 */
@Component
public class QiNiuUtil {
    @Value("${qiNiu.url}")
    public String url;
    public String getUrl() {
        return url;
    }

    @Value("${qiNiu.accessKey}")
    private String accessKey;
    @Value("${qiNiu.accessSecretKey}")
    private String accessSecretKey;

    @Value("${qiNiu.bucketName}")
    private String bucket;

    public boolean upload(MultipartFile file,String fileName) {
        //构造一个带指定 Region 对象的配置类
        Configuration cfg = new Configuration(Region.autoRegion());
        //...其他参数参考类注释
        UploadManager uploadManager = new UploadManager(cfg);
        //...生成上传凭证，然后准备上传

        //默认不指定key的情况下，以文件内容的hash值作为文件名
        try {
            byte[] uploadBytes = file.getBytes();
            Auth auth = Auth.create(accessKey, accessSecretKey);
            String upToken = auth.uploadToken(bucket);
            Response response = uploadManager.put(uploadBytes, fileName, upToken);
            //解析上传成功的结果
            DefaultPutRet putRet = JSON.parseObject(response.bodyString(), DefaultPutRet.class);
            return true;
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return false;
    }
}
```



### 后端代码

```java
package com.lee.uploadfile.controller;

import com.lee.uploadfile.util.QiNiuUtil;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import javax.annotation.Resource;
import java.util.UUID;

/**
 * 将图片上传至七牛云
 * @author 晓龙coding
 */
@RestController
@RequestMapping("/upload")
public class UploadController {
    @Resource
    private QiNiuUtil qiNiuUtil;

    @PostMapping("/cloud")
    String uploadPictureCloud(MultipartFile file) {
        // 截取文件名，并用新的文件名替换
        String originalFilename = file.getOriginalFilename();
        assert originalFilename != null;
        int startIndex = originalFilename.lastIndexOf('.');
        String ext = originalFilename.substring(startIndex);
        String randomName = UUID.randomUUID().toString().replace("-", "");
        String newFilename = randomName + ext;
        
        // 判断是否上传成功
        boolean success = qiNiuUtil.upload(file, newFilename);
        if (success) {
            return qiNiuUtil.getUrl() + newFilename;
        }
        // 图片上传失败，返回空串
        return "";
    }
}
```



### 前端代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form action="http://localhost:8080/upload/cloud" method="post" enctype="multipart/form-data">
        <!-- name 属性用于对提交到服务器后的表单数据进行标识 -->
        <input type="file" name="file" value="上传图片">
        <input type="submit" value="上传">
    </form>
</body>
</html>
```





## 一些坑

### 使用 apifox/postman 测试

注意：参数名需要是file才可以上传成功。

![QQ截图20230130201059](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/typora/QQ%E6%88%AA%E5%9B%BE20230130201059.png)



### 返回值需要是字典对象

有时候前端只能接收字典类型的对象。
