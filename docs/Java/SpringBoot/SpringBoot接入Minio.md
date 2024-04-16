# SpringBoot接入Minio

[toc]



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



#### 1. 配置依赖

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



#### 2. application.yml配置信息

```YAML
minio:
  endpoint: api.minio.lihailong.vip
  bucketName: picture
  accessKey: GGaVLCgUNefF7y5HsfsR
  secretKey: M7JGgaDmmB8kQko1JUsjbw7KgOOKeu4NzRnhZYYl
```



#### 3. 配置类

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



#### 4. 工具类

使用nginx做反向代理时，如果bucketExist这个api出错，记得给nginx的配置设置：`proxy_cache_convert_head off`

```Java
package com.lhl.minio_demo.util;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.util.IdUtil;
import cn.hutool.core.util.StrUtil;
import com.lhl.minio_demo.config.MinioConfig;
import com.lhl.minio_demo.exception.BizException;
import com.lhl.minio_demo.exception.SysException;
import io.micrometer.common.util.StringUtils;
import io.minio.*;
import io.minio.errors.*;
import jakarta.annotation.Resource;
import jakarta.servlet.ServletOutputStream;
import jakarta.servlet.http.HttpServletResponse;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;
import org.springframework.util.FastByteArrayOutputStream;
import org.springframework.web.multipart.MultipartFile;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.List;
import java.util.Objects;


@Component
@Slf4j
public class MinioUtil implements InitializingBean {
    @Resource
    private MinioConfig prop;

    @Resource
    private MinioClient minioClient;

    /**
     * 项目启动时校验
     */
    @Override
    public void afterPropertiesSet() {
        boolean bucketExist = bucketExist(prop.getBucketName());
        if (bucketExist) {
            log.info("对象桶存在, 桶名:{}，minio加载成功。", prop.getBucketName());
        } else {
            log.error("对象桶不存在, 桶名:{}", prop.getBucketName());
        }
    }

    /**
     * 判断桶是否存在
     * @param bucketName
     * @return
     */
    public boolean bucketExist(String bucketName) {
        try {
            return minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());
        } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException |
                 InvalidResponseException | IOException | NoSuchAlgorithmException | ServerException |
                 XmlParserException e) {
            return false;
        }
    }

    /**
     * 判断对象是否存在
     * @param filename
     * @return
     * @throws SysException
     */
    public boolean objectExist(String filename) throws SysException {
        try {
            GetObjectResponse response = minioClient.getObject(GetObjectArgs.builder().bucket(prop.getBucketName()).object(filename).build());
            return true;
        } catch (ErrorResponseException | InsufficientDataException | InternalException | InvalidKeyException |
                 InvalidResponseException | IOException | NoSuchAlgorithmException | ServerException |
                 XmlParserException e) {
            return false;
        }
    }

    /**
     * 判断上传的文件类型是否有效
     * @param contentType
     * @return
     */
    public boolean fileContentTypeValid(String contentType) {
        for (ContentTypeEnums value : ContentTypeEnums.values()) {
            if (Objects.equals(value.getContentType(), contentType)) {
                return true;
            }
        }
        return false;
    }

    /**
     * 文件上传，通常应用与spring框架中，前端将文件直接传给后端
     * @param file 上传的文件
     * @return 上传后的文件地址
     * @throws SysException 系统异常
     * @throws BizException 业务异常
     */
    public UploadFileResponse upload(MultipartFile file, UploadParam uploadParam) throws SysException, BizException, IOException {
        return upload(file.getBytes(), file.getOriginalFilename(), file.getContentType(), uploadParam);
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
    public UploadFileResponse upload(byte[] fileBytes, String originalFilename, String contentType, UploadParam uploadParam) throws SysException, BizException {
        if (! fileContentTypeValid(contentType)) {
            throw new BizException("文件上传类型不支持，当前contentType:" + contentType);
        }
        if (fileBytes.length == 0) {
            throw new BizException("文件不能为空");
        }
        if (StringUtils.isBlank(originalFilename)) {
            throw new BizException("文件名不能为空");
        }

        // 构造文件的前缀
        // 在minio中，如果出现 / 则会自动构建目录，因此我们只需要将目录名和 / 拼接在文件名中即可
        String fileName;
        if (uploadParam.getChangeFileName() == UploadParam.DONT_CHANGE_FILENAME) {
            fileName = originalFilename;
        } else if (uploadParam.getChangeFileName() == UploadParam.CHANGE_FILENAME) {
            fileName = DateUtil.format(DateUtil.date(), "yyyy-MM-dd") + IdUtil.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
        } else {
            throw new BizException("文件名称修改策略异常，文件名称修改参数:" + uploadParam.getChangeFileName());
        }

        String fullPathFileName = buildFilePath(uploadParam.getPrefix(), fileName);

        // 判断文件覆盖策略
        boolean updateFile = false;
        if (uploadParam.getOverrideType() == UploadParam.OVERRIDE) {
            updateFile = true;
        } else if (uploadParam.getOverrideType() == UploadParam.DONT_OVERRIDE) {
            // 先检查文件是否存在
            boolean exist = objectExist(fullPathFileName);
            if (exist) {
                updateFile = false;
            } else {
                updateFile = true;
            }
        } else {
            throw new BizException(StrUtil.format("上传策略异常，上传策略类型:{}\n", uploadParam.getOverrideType()));
        }

        UploadFileResponse uploadFileResponse = new UploadFileResponse();
        if (updateFile) {
            try {
                PutObjectArgs objectArgs = PutObjectArgs.builder()
                        .bucket(prop.getBucketName())
                        .object(fullPathFileName)
                        .stream(new ByteArrayInputStream(fileBytes), fileBytes.length, -1)
                        .contentType(contentType)
                        .build();
                minioClient.putObject(objectArgs);
                String url = prop.getEndpoint() + "/" + prop.getBucketName() + "/" + fullPathFileName;
                uploadFileResponse.setUpdateSuccess(true);
                uploadFileResponse.setUrl(url);
            } catch (Exception e) {
                throw new SysException("上传文件时，系统异常", e);
            }
        } else {
            uploadFileResponse.setUpdateSuccess(false);
            uploadFileResponse.setMsg(fileName);
        }

        return uploadFileResponse;
    }

    private String buildFilePath(String[] prefix, String filename) {
        if (CollectionUtils.isEmpty(List.of(prefix))) {
            return filename;
        }
        StringBuilder sb = new StringBuilder();
        for (String dir : prefix) {
            sb.append(dir).append("/");
        }
        sb.append(filename);
        return sb.toString();
    }

    /**
     * 文件下载
     * @param fileName 文件名称
     * @param res response
     */
    public void download(String fileName, String[] prefix, HttpServletResponse res) throws SysException {
        String fullPathFileName = buildFilePath(prefix, fileName);
        GetObjectArgs objectArgs = GetObjectArgs.builder().bucket(prop.getBucketName())
                .object(fullPathFileName).build();
        try (GetObjectResponse response = minioClient.getObject(objectArgs)) {
            byte[] buf = new byte[1024];
            int len;
            try (FastByteArrayOutputStream os = new FastByteArrayOutputStream()) {
                while ((len=response.read(buf)) != -1){
                    os.write(buf, 0, len);
                }
                os.flush();
                byte[] bytes = os.toByteArray();
                res.setCharacterEncoding(StandardCharsets.UTF_8.name());
                // 设置强制下载不打开
                res.setContentType("application/force-download");
                res.setContentType("application/octet-stream");
                // 防止中文编码异常。如果出现中文编码异常，那么浏览器中就不能直接下载pdf文件，而是展示pdf文件。
                res.addHeader("Content-Disposition", "attachment;fileName=" + URLEncoder.encode(fileName, StandardCharsets.UTF_8.name()));
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
            String targetFilename = CommonConstant.PAPER_RESOURCE_PREFIX + filename;
            GetObjectResponse objectResponse = minioClient.getObject(
                    GetObjectArgs.builder()
                            .bucket(prop.getBucketName())
                            .object(targetFilename)
                            .build());
            return IoUtil.readBytes(objectResponse);
        } catch (Exception e) {
            throw new SysException("获取文件流异常", e);
        }
    }

    @Data
    public static class UploadParam {
        /**
         * 0: 重命名文件
         * 1: 保持原来的文件名
         */
        private final int changeFileName;

        public static final int CHANGE_FILENAME = 0;
        public static final int DONT_CHANGE_FILENAME = 1;

        /**
         * 遇到同名文件时
         * 0: 抛出异常
         * 1: 不更新文件
         * 2: 覆盖原文件
         */
        private final int overrideType;

        public static final int DONT_OVERRIDE = 0;
        public static final int OVERRIDE = 1;

        /**
         * 文件可能放在多级目录下，此时需要将每一级的目录名都放在prefix数组中
         */
        private final String[] prefix;

        public UploadParam(int changeFileName, int overrideType, String[] prefix) {
            this.changeFileName = changeFileName;
            this.overrideType = overrideType;
            this.prefix = prefix;
        }
    }

    @Data
    public static class UploadFileResponse {
        // 是否成功上传文件
        private boolean updateSuccess;

        // 上传成功文件的url。失败时为空。
        private String url;

        // 上传失败的文件名。成功时为空
        private String msg;
    }

    public enum ContentTypeEnums {
        PDF("application/pdf"),
        WORD("application/msword"),
        PNG("image/png"),
        JPG("image/jpg"),
        JPEG("image/jpeg");

        private final String contentType;

        ContentTypeEnums(String contentType) {
            this.contentType = contentType;
        }

        public String getContentType() {
            return this.contentType;
        }
    }
}

```



#### 5. 使用案例

上传

```Java
package com.lhl.minio_demo.controller;

import com.lhl.minio_demo.exception.BizException;
import com.lhl.minio_demo.exception.SysException;
import com.lhl.minio_demo.util.MinioUtil;
import jakarta.annotation.Resource;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

@RestController
@RequestMapping("upload")
public class UploadController {
    @Resource
    private MinioUtil minioUtil;

    @PostMapping("picture")
    public MinioUtil.UploadFileResponse uploadPicture(MultipartFile file) throws SysException, BizException, IOException {
        MinioUtil.UploadParam uploadParam = new MinioUtil.UploadParam(
                MinioUtil.UploadParam.DONT_CHANGE_FILENAME,
                MinioUtil.UploadParam.DONT_OVERRIDE,
                new String[]{"first-dir", "second-dir", "last-dir"});  // 文件名中不建议带/，否则会被拆分为多级目录
        MinioUtil.UploadFileResponse uploadFileResponse = minioUtil.upload(file, uploadParam);
        return uploadFileResponse;
    }
}

```

下载

```java
package com.lhl.minio_demo.controller;

import com.lhl.minio_demo.exception.SysException;
import com.lhl.minio_demo.util.MinioUtil;
import jakarta.annotation.Resource;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("download")
public class DownloadController {
    @Resource
    private MinioUtil minioUtil;
    @Resource
    private HttpServletResponse httpServletResponse;

    @RequestMapping
    public String download(@RequestParam("name") String name) throws SysException {
        minioUtil.download(name, new String[]{"first-dir", "second-dir", "last-dir"}, httpServletResponse);
        return "下载成功";
    }
}

```

