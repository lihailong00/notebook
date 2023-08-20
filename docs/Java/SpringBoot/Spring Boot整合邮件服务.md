# Spring Boot整合邮件服务

[toc]



## 基本概念

1. 抄送与密送是什么？



## 使用Spring Boot自带的邮件服务

### 步骤解析

1. 开启QQ邮箱的IMAP/SMTP服务，百度搜索教程即可。
2. 创建Spring Boot项目，引入相关依赖，并在`application.properties`文件中加入配置信息。
3. 自定义邮件发送工具类并使用。



### 代码

> pom依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
	<!--邮件工具类-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```



>application.properties配置文件

```properties
spring.mail.host=smtp.qq.com
spring.mail.username=发件者邮箱
spring.mail.password=qq邮件服务器开启imap协议后生成的密码
spring.mail.protocol=smtps
spring.mail.properties.mail.smtp.ssl.enable=true
```



> 自定义邮件类

```java
package com.lee.easymail.pojo;

import lombok.Data;
import org.springframework.core.io.ByteArrayResource;

import java.io.File;
import java.io.InputStream;
import java.util.Map;

/**
 * @author 晓龙coding
 */
@Data
public class MyMail {
    /**
     * 邮件接收人
     */
    private String[] receiver;
    /**
     * 邮件主题
     */
    private String subject;
    /**
     * 邮件的文本内容
     */
    private String content;
    /**
     * 抄送人
     */
    private String[] cc;
    /**
     * 密送人
     */
    private String[] bcc;
    /**
     * 邮件附件的文件名
     */
    private String attachFileName;
    /**
     * 附件内容，可以是byte[]类型，可以是File类型
     * 如果文件在数据库中，则使用byte[]
     * 如果文件在服务器上，则使用File
     */
    private ByteArrayResource attachment;
    /**
     * 邮件内容内嵌图片
     */
    private Map<String, String> imageMap;
    /**
     * 是否是html文本
     */
    private boolean html = false;
}
```





> 邮件发送工具类

```java
package com.lee.easymail.util;

import com.lee.easymail.pojo.MyMail;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;

/**
 * @author 晓龙coding
 */
@Component
@Slf4j
public class MyMailSender {
    @Value("${spring.mail.username}")
    private String from;
    @Resource
    private JavaMailSender mailSender;

    /**
     * 发送邮件
     * @param mail 邮件对象
     */
    public void send(MyMail mail) {
        try {
            MimeMessage message = mailSender.createMimeMessage();
            // 这里设置为true才能发送附件
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from);
            helper.setTo(mail.getReceiver());
            helper.setSubject(mail.getSubject());
            //第二个参数为true表示邮件正文是html格式的，默认是false
            helper.setText(mail.getContent(), mail.isHtml());
            if (mail.getCc() != null) {
                helper.setCc(mail.getCc());
            }
            if (mail.getBcc() != null) {
                helper.setBcc(mail.getBcc());
            }
            // 添加附件
            if (mail.getAttachment() != null) {
                helper.addAttachment(mail.getAttachFileName(), mail.getAttachment());
            }
            mailSender.send(helper.getMimeMessage());
        } catch (MessagingException e) {
            log.info("邮件发送失败~");
        }
    }
}
```



> 调用工具类：通过注解，从IOC容器中获取工具类对象

```java
import com.lee.easymail.pojo.MyMail;
import com.lee.easymail.pojo.Paper;
import com.lee.easymail.util.MyMailSender;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.core.io.ByteArrayResource;

import javax.annotation.Resource;

@Resource
private MyMailSender myMailSender;

public String sendMail() {
    MyMail mail = new MyMail();
    // 设置接收方
    mail.setReceiver(new String[]{"aaa@qq.com"});
    // 设置抄送地址
    mail.setCc(new String[]{"bbb@qq.com", "ccc@qq.com"});
    // 设置密送地址
    mail.setBcc(new String[]{"ddd@qq.com"});
    mail.setSubject("这是我的测试主题");
    mail.setContent("这是我的测试内容");

    // 获取附件内容
    // 假定Paper类的结构是这样的：{ String, paperName, byte[] paperContent }
    Paper paper = paperService.getPaperById(123);
    if (paper != null) {
        String paperName = paper.getPaperName();
        byte[] paperContent = paper.getPaperContent();
        // 设置附件名
        mail.setAttachFileName(paperName);
        // 设置附件内容
        mail.setAttachment(new ByteArrayResource(paperContent));
    }
    myMailSender.send(mail);
    return "发送成功！";
}
```





## 整合阿里云邮件服务

### 前置要求



### 官方文档

[快速使用API和SMTP发信的流程简化说明 (aliyun.com)](https://help.aliyun.com/document_detail/430694.html?spm=a2c4g.11186623.0.0.620e25f4H1FgzI)：大致掌握发信流程。

[发送邮件的配置步骤简化说明 (aliyun.com)](https://help.aliyun.com/document_detail/328553.html?spm=a2c4g.11186623.0.0.734b3e068NAZAX)：需要有一个备案的域名。

[SMTP 之 Java 调用示例 (aliyun.com)](https://help.aliyun.com/document_detail/29450.html)：我将代码封装成一个工具，在后续文档中。



### 代码

导入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!--邮件类依赖-->
    <dependency>
        <groupId>com.sun.mail</groupId>
        <artifactId>javax.mail</artifactId>
        <version>1.6.2</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.26</version>
        <scope>provided</scope>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--mysql驱动-->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.0.32</version>
    </dependency>

    <!--mybatis-plus-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.3.1</version>
    </dependency>
</dependencies>
```



配置信息：配置发信地址，SMTP密码和回信地址。在邮件控制台中可以查看。

```properties
ali.simple-mail.account=aaa@bbb.com
ali.simple-mail.password=123456
ali.simple-mail.reply-address=ccc@ddd.com
```



邮件实体类：

```java
package com.lee.mymail2.pojo;

import lombok.Data;
import org.springframework.core.io.ByteArrayResource;

import javax.mail.util.ByteArrayDataSource;
import java.util.Map;

/**
 * @author 晓龙coding
 */
@Data
public class MyMail {
    /**
     * 发件人姓名
     */
    private String sender;
    /**
     * 邮件接收人
     */
    private String[] receiver;
    /**
     * 邮件主题
     */
    private String subject;
    /**
     * 邮件的文本内容
     */
    private String content;
    /**
     * 抄送人：用逗号隔开不同的抄送人
     */
    private String[] cc;
    /**
     * 密送人：用逗号隔开不同的密送人
     */
    private String[] bcc;
    /**
     * 邮件附件的文件名
     */
    private String attachFileName;
    /**
     * 附件内容，可以是byte[]类型，可以是File类型
     * 如果文件在数据库中，则使用byte[]
     * 如果文件在服务器上，则使用File
     */
    private ByteArrayDataSource attachment;
    /**
     * 邮件内容内嵌图片
     */
    private Map<String, String> imageMap;
    /**
     * 文本格式：默认为纯文本
     * 纯文本：text/plain;charset=UTF-8
     * 超文本：text/html;charset=UTF-8
     */
    private String contentType = "text/plain;charset=UTF-8";
}
```

邮件工具类：

```java
package com.lee.mymail2.util;

import com.lee.mymail2.pojo.MyMail;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.activation.DataHandler;
import javax.mail.*;
import javax.mail.internet.*;
import java.io.UnsupportedEncodingException;
import java.time.LocalDateTime;
import java.util.Date;
import java.util.Properties;

/**
 * @author 晓龙coding
 */
@Component
@Slf4j
public class AliMailUtil {
    @Value("${ali.simple-mail.account}")
    private String account;
    @Value("${ali.simple-mail.password}")
    private String password;
    @Value("${ali.simple-mail.reply-address}")
    private String reply;
    public boolean sendMimeMail(MyMail mail) {
        // 配置发送邮件的环境属性
        final Properties props = new Properties();

        // 表示SMTP发送邮件，需要进行身份验证
        props.put("mail.smtp.auth", "true");
        props.put("mail.smtp.host", "smtpdm.aliyun.com");
        //设置端口：
        props.put("mail.smtp.port", "80");//或"25", 如果使用ssl，则去掉使用80或25端口的配置，进行如下配置：
        //加密方式：
        //props.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
        //props.put("mail.smtp.socketFactory.port", "465");
        //props.put("mail.smtp.port", "465");

        props.put("mail.smtp.from", account);    //mailfrom 参数
        props.put("mail.user", account);// 发件人的账号（在控制台创建的发信地址）
        props.put("mail.password", password);// 发信地址的smtp密码（在控制台选择发信地址进行设置）
        //props.put("mail.smtp.connectiontimeout", 1000);

        // 构建授权信息，用于进行SMTP进行身份验证
        Authenticator authenticator = new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                // 用户名、密码
                String userName = props.getProperty("mail.user");
                String password = props.getProperty("mail.password");
                return new PasswordAuthentication(userName, password);
            }
        };

        //使用环境属性和授权信息，创建邮件会话
        Session mailSession = Session.getInstance(props, authenticator);
        //mailSession.setDebug(true);//开启debug模式

        //UUID uuid = UUID.randomUUID();
        //final String messageIDValue = "<" + uuid.toString() + ">";
        //创建邮件消息
        MimeMessage message = new MimeMessage(mailSession) {
            //@Override
            //protected void updateMessageID() throws MessagingException {
            //设置自定义Message-ID值
            //setHeader("Message-ID", messageIDValue);//创建Message-ID
            //}
        };

        try {
            // 设置发件人邮件地址和名称。填写控制台配置的发信地址。和上面的mail.user保持一致。名称用户可以自定义填写。
            InternetAddress from = new InternetAddress(account, mail.getSender());//from 参数,可实现代发，注意：代发容易被收信方拒信或进入垃圾箱。
            message.setFrom(from);

            // 可选。设置回信地址
            Address[] a = new Address[1];
            a[0] = new InternetAddress(reply);
            message.setReplyTo(a);

            // 设置收件人邮件地址
            // 同时发给多人（因为部分收信系统的一些限制，尽量每次投递给一个人；同时我们限制单次允许发送的人数是60人）：
            InternetAddress[] adds = new InternetAddress[mail.getReceiver().length];
            for (int i = 0; i < mail.getReceiver().length; i++) {
                adds[i] = new InternetAddress(mail.getReceiver()[i]);
            }
            message.setRecipients(Message.RecipientType.TO, adds);
            message.setSentDate(new Date()); //设置时间
            // 设置邮件主题
            message.setSubject(mail.getSubject());

//            message.setContent(mail.getContent(), mail.getContentType());


            // 设置多个抄送地址
            if (null != mail.getCc()) {
                StringBuilder ccUser = new StringBuilder();
                for (int i = 0; i < mail.getCc().length; i++) {
                    ccUser.append(mail.getCc()[i]);
                    if (i != mail.getCc().length - 1) {
                        ccUser.append(",");
                    }
                }
                @SuppressWarnings("static-access")
                InternetAddress[] internetAddressCC = new InternetAddress().parse(String.valueOf(ccUser));
                message.setRecipients(Message.RecipientType.CC, internetAddressCC);
            }

            // 设置多个密送地址
            if (null != mail.getBcc()) {
                StringBuilder bccUser = new StringBuilder();
                for (int i = 0; i < mail.getBcc().length; i++) {
                    bccUser.append(mail.getBcc()[i]);
                    if (i != mail.getBcc().length - 1) {
                        bccUser.append(",");
                    }
                }
                @SuppressWarnings("static-access")
                InternetAddress[] internetAddressBCC = new InternetAddress().parse(String.valueOf(bccUser));
                message.setRecipients(Message.RecipientType.BCC, internetAddressBCC);
            }

//            //若需要开启邮件跟踪服务，请使用以下代码设置跟踪链接头。前置条件和约束见文档"如何开启数据跟踪功能？"
//            String tagName = "Test";
//            HashMap<String, String> trace = new HashMap<>();
//            //这里为字符串"1"
//            trace.put("OpenTrace", "1");      //打开邮件跟踪
//            trace.put("LinkTrace", "1");     //点击邮件里的URL跟踪
//            trace.put("TagName", tagName);   //控制台创建的标签tagname
//            String jsonTrace = new GsonBuilder().setPrettyPrinting().create().toJson(trace);
//            //System.out.println(jsonTrace);
//            String base64Trace = new String(Base64.getEncoder().encode(jsonTrace.getBytes()));
//            //设置跟踪链接头
//            message.addHeader("X-AliDM-Trace", base64Trace);
//            //邮件eml原文中的示例值：X-AliDM-Trace: eyJUYWdOYW1lIjoiVGVzdCIsIk9wZW5UcmFjZSI6IjEiLCJMaW5rVHJhY2UiOiIxIn0=

            //发送附件和内容：
            BodyPart messageBodyPart = new MimeBodyPart();
            //messageBodyPart.setText("消息<br>Text");//设置邮件的内容，文本
            messageBodyPart.setContent(mail.getContent(), mail.getContentType());// 纯文本："text/plain;charset=UTF-8" //设置邮件的内容
            //创建多重消息
            Multipart multipart = new MimeMultipart();
            //设置文本消息部分
            multipart.addBodyPart(messageBodyPart);

            //附件部分

            if (mail.getAttachment() != null) {
                //发送附件，总的邮件大小不超过15M，创建消息部分。
                MimeBodyPart mimeBodyPart = new MimeBodyPart();
                //设置要发送附件的文件路径
                mimeBodyPart.setDataHandler(new DataHandler(mail.getAttachment()));
                if (mail.getAttachFileName() == null) {
                    mail.setAttachFileName("默认文件");
                    log.info("============start============");
                    log.info("重要异常");
                    log.info("文件存在，但是文件名不存在。请查看数据库数据的完整性。");
                    log.info("============start============\n");
                }
                //处理附件名称中文（附带文件路径）乱码问题
                mimeBodyPart.setFileName(MimeUtility.encodeText(mail.getAttachFileName(), "UTF-8", "B"));
                mimeBodyPart.addHeader("Content-Transfer-Encoding", "base64");
                multipart.addBodyPart(mimeBodyPart);
            }

            //发送含有附件的完整消息
            message.setContent(multipart);
            // 发送附件代码，结束

            // 发送邮件
            Transport.send(message);

        } catch (MessagingException e) {
            String err = e.getMessage();
            // 在这里处理message内容， 格式是固定的
            log.info("============start============");
            log.info("重要异常，邮件发送异常");
            log.info("发生时间: {}", LocalDateTime.now());
            log.info("异常详情: {}", err);
            log.info("============end============");
        } catch (UnsupportedEncodingException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            return false;
        }
        return true;
    }
}

```

实战：

```java
@Resource
private PaperService paperService;
@Resource
private AliMailUtil aliMailUtil;
@RequestMapping("")
public String sendMail() {
    MyMail mail = new MyMail();
    mail.setSender("晓龙coding");
    mail.setSubject("======标题======");
    mail.setContent("这是测试内容，hello world");
    mail.setReceiver(new String[]{"aaa@bbb.com"});
    mail.setCc(new String[]{"ccc@ddd.com", "eee@fff.com"});
    mail.setBcc(new String[]{"ggg@hhh.com", "iii@jjj.com"});
    // 也可以设置为HTML格式
    mail.setContentType("text/plain;charset=UTF-8");
    // 假定paper的类型是{byte[] paperContent; String paperName; String paId;}
    Paper paper = paperService.getPaperById(123L);
    if (paper != null) {
        mail.setAttachment(new ByteArrayDataSource(paper.getPaperContent(), "application/octet-stream"));
        mail.setAttachFileName(paper.getPaperName());
    }
    boolean success = aliMailUtil.sendMimeMail(mail);
    if (success) {
        return "发送成功！";
    }
    return "发送失败~";
}
```

