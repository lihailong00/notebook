# SpringBoot接入Minio



## 环境

1. JDK 21
2. SpringBoot 3.2.0



## 步骤

### 一、安装Minio

建议直接去1panel上快捷安装Minio镜像。

### 二、给Minio服务配置域名

1. 建议先在自己的服务器上安装openrestry（建议1panel上快捷安装）。
2. Minio服务有两个接口，默认9000是操作界面的接口，9001是api接口（可以随意定义接口）。然后腾讯云DNS申请两个域名，解析到自己的服务器的80端口。
3. OpenRestry中配置反向代理，分别代理操作页面对应的接口和api对应的接口。
4. 尽量给两个域名都申请SSL证书（建议在1panel上快捷申请）。

### 三、初始化Spring Boot项目

代码中抛出异常的部分可以根据自己的情况做处理。

1. idea中使用Spring initializr初始化最新版的springboot。
2. 配置依赖

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.lhl</groupId>
    <artifactId>minio_demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>minio_demo</name>
    <description>minio_demo</description>
    <properties>
        <java.version>21</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>io.minio</groupId>
            <artifactId>minio</artifactId>
            <version>8.5.7</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/cn.hutool/hutool-all -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.8.23</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>RELEASE</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

3. application.yml配置信息

```YAML
minio:
  endpoint: api.minio.lihailong.vip
  bucketName: picture
  accessKey: GGaVLCgUNefF7y5HsfsR
  secretKey: M7JGgaDmmB8kQko1JUsjbw7KgOOKeu4NzRnhZYYl
```

4. 配置类

```Java
package com.lhl.minio_demo.config;

import io.minio.MinioClient;
import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Data
@Configuration
public class MinioConfig {
    @Value("${minio.endpoint}")
    private String endpoint;
    @Value("${minio.accessKey}")
    private String accessKey;
    @Value("${minio.secretKey}")
    private String secretKey;
    @Value("${minio.bucketName}")
    private String bucketName;

    @Bean
    public MinioClient minioClient() {
        return MinioClient.builder()
                .endpoint(endpoint)
                .credentials(accessKey, secretKey)
                .build();
    }
}
```

5. 工具类

```Java
package com.lhl.minio_demo.util;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.io.IoUtil;
import cn.hutool.core.util.IdUtil;
import com.lhl.minio_demo.config.MinioConfig;
import com.lhl.minio_demo.exception.BizException;
import com.lhl.minio_demo.exception.SysException;
import io.micrometer.common.util.StringUtils;
import io.minio.GetObjectArgs;
import io.minio.GetObjectResponse;
import io.minio.MinioClient;
import io.minio.PutObjectArgs;
import jakarta.annotation.Resource;
import jakarta.servlet.ServletOutputStream;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.util.FastByteArrayOutputStream;
import org.springframework.web.multipart.MultipartFile;

import java.io.ByteArrayInputStream;

@Component
@Slf4j
public class MinioUtil {
    @Resource
    private MinioConfig prop;

    @Resource
    private MinioClient minioClient;

    /**
     * 文件上传，通常应用与spring框架中，前端将文件直接传给后端
     * @param file 上传的文件
     * @return 上传后的文件地址
     * @throws SysException 系统异常
     * @throws BizException 业务异常
     */
    public String upload(MultipartFile file) throws SysException, BizException {
        if (file == null || file.isEmpty()) {
            throw new BizException("文件不能为空");
        }
        String originalFilename = file.getOriginalFilename();
        if (StringUtils.isBlank(originalFilename)) {
            throw new BizException("文件名不能为空");  // 这里也可以返回空
        }

        String fileName = IdUtil.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
        String objectName = DateUtil.format(DateUtil.date(), "yyyy-MM-dd") + "+" + fileName;
        try {
            PutObjectArgs objectArgs = PutObjectArgs.builder()
                    .bucket(prop.getBucketName())
                    .object(objectName)
                    .stream(file.getInputStream(), file.getSize(), -1)
                    .contentType(file.getContentType())
                    .build();
            //文件名称相同会覆盖
            minioClient.putObject(objectArgs);
        } catch (Exception e) {
            throw new SysException("上传文件时，系统异常", e);
        }
        String url = prop.getEndpoint() + "/" + prop.getBucketName() + "/" + objectName;
        return url;
    }

    /**
     * 文件上传，这种方式通常应用于后端上传文件。因为后端很难获取 MultipartFile
     * @param fileBytes 文件的字节流
     * @param originalFilename 文件名称
     * @param contentType 文件类型，例如image/png  image/jpg
     * @return 上传后的文件地址
     * @throws SysException
     * @throws BizException
     */
    public String upload(byte[] fileBytes, String originalFilename, String contentType) throws SysException, BizException {
        if (fileBytes.length == 0) {
            throw new BizException("文件不能为空");
        }
        if (StringUtils.isBlank(originalFilename)) {
            throw new BizException("文件名不能为空");
        }
        String fileName = IdUtil.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
        String objectName = DateUtil.format(DateUtil.date(), "yyyy-MM-dd") + "+" + fileName;
        try {
            PutObjectArgs objectArgs = PutObjectArgs.builder()
                    .bucket(prop.getBucketName())
                    .object(objectName)
                    .stream(new ByteArrayInputStream(fileBytes), fileBytes.length, -1)
                    .contentType(contentType)
                    .build();
            //文件名称相同会覆盖
            minioClient.putObject(objectArgs);
        } catch (Exception e) {
            throw new SysException("上传文件时，系统异常", e);
        }
        String url = prop.getEndpoint() + "/" + prop.getBucketName() + "/" + objectName;
        return url;
    }

    /**
     * 文件下载
     * @param fileName 文件名称
     * @param res response
     */
    public void download(String fileName, HttpServletResponse res) throws SysException {
        GetObjectArgs objectArgs = GetObjectArgs.builder().bucket(prop.getBucketName())
                .object(fileName).build();
        try (GetObjectResponse response = minioClient.getObject(objectArgs)) {
            byte[] buf = new byte[1024];
            int len;
            try (FastByteArrayOutputStream os = new FastByteArrayOutputStream()) {
                while ((len=response.read(buf)) != -1){
                    os.write(buf, 0, len);
                }
                os.flush();
                byte[] bytes = os.toByteArray();
                res.setCharacterEncoding("utf-8");
                // 设置强制下载不打开
                // res.setContentType("application/force-download");
                res.addHeader("Content-Disposition", "attachment;fileName=" + fileName);
                try (ServletOutputStream stream = res.getOutputStream()){
                    stream.write(bytes);
                    stream.flush();
                }
            }
        } catch (Exception e) {
            throw new SysException("下载异常", e);
        }
    }

    /**
     * 获取文件流
     *
     * @param filename 文件名
     * @return 二进制流
     */
    public byte[] getObjectByteArray(String filename) throws SysException {
        try {
            GetObjectResponse objectResponse = minioClient.getObject(
                    GetObjectArgs.builder()
                            .bucket(prop.getBucketName())
                            .object(filename)
                            .build());
            return IoUtil.readBytes(objectResponse);
        } catch (Exception e) {
            throw new SysException("获取文件流异常", e);
        }
    }
}
```



使用案例：

```Java
@RestController
@RequestMapping("")
public class FileController {
    @Resource
    private MinioUtil minioUtil;
    @Resource
    private HttpServletResponse httpServletResponse;

    // 上传一个文件（主要是图片）
    @PostMapping("upload")
    public String uploadPicture(MultipartFile file) throws SysException, BizException {  // 可以删掉我的自定义异常
        return minioUtil.upload(file);
    }

    // 下载一个文件
    @RequestMapping("download")
    public String download(@RequestParam("filename") String fileName) throws SysException {
        minioUtil.download(fileName, httpServletResponse);
        return "下载成功";
    }
}
```
