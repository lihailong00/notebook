# TraceId链路追踪



[toc]



## 依赖

```xml
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
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
    </dependency>

    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.8.25</version>
    </dependency>

    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.14.0</version>
    </dependency>
  
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
```



## 返回结构体

```java
package com.lee.traceiddemo.model.dto;

import com.lee.traceiddemo.util.TraceIdUtil;
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

    /**
     * 链路追踪traceId
     */
    public String traceId;

    public static ResponseResult success() {
        return new ResponseResult(200, true, "响应正常", null, TraceIdUtil.getTraceId());
    }

    public static ResponseResult success(Object data) {
        return new ResponseResult(200, true, "响应正常", data, TraceIdUtil.getTraceId());
    }

    public static ResponseResult success(Object data, String msg) {
        return new ResponseResult(200, true, msg, data, TraceIdUtil.getTraceId());
    }

    public static ResponseResult fail(String msg) {
        return new ResponseResult(200, false, msg, null, TraceIdUtil.getTraceId());
    }

    public static ResponseResult fail(String msg, Object data) {
        return new ResponseResult(200, false, msg, data, TraceIdUtil.getTraceId());
    }
}

```



## traceId工具类

```java
package com.lee.traceiddemo.util;

import cn.hutool.core.util.IdUtil;
import io.micrometer.common.util.StringUtils;
import org.slf4j.MDC;

public class TraceIdUtil {
    public static final String TRACE_ID_KEY = "TraceId";
    public static String generateTraceId() {
        String traceId = IdUtil.fastSimpleUUID().toUpperCase();
        MDC.put(TRACE_ID_KEY, traceId);
        return traceId;
    }

    public static String generateTraceId(String traceId) {
        if (StringUtils.isBlank(traceId)) {
            return generateTraceId();
        }
        MDC.put(TRACE_ID_KEY, traceId);
        return traceId;
    }

    public static String getTraceId() {
        return MDC.get(TRACE_ID_KEY);
    }

    public static void removeTraceId() {
        MDC.remove(TRACE_ID_KEY);
    }
}

```



## 拦截器配置

```java
package com.lee.traceiddemo.filter;


import cn.hutool.core.util.IdUtil;
import com.lee.traceiddemo.util.TraceIdUtil;
import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import java.io.IOException;

@Slf4j
public class TraceFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("Init Trace filter......");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        try {
            HttpServletRequest request = (HttpServletRequest) servletRequest;
            // 请求进来后，查看请求头是否带有traceId
            // 大多数情况下，请求刚进来不会携带traceId
            // 但是有时候可以在网关携带traceId
            String traceId = ((HttpServletRequest) servletRequest).getHeader(TraceIdUtil.TRACE_ID_KEY);
            if (StringUtils.isBlank(traceId)) {
                traceId = IdUtil.fastSimpleUUID().toUpperCase();
            }
            TraceIdUtil.generateTraceId(traceId);
            filterChain.doFilter(servletRequest, servletResponse);
        } catch (Exception e) {
            log.error("traceId异常: {}", e.getMessage(), e);
        } finally {
            // 防止内存溢出
            TraceIdUtil.removeTraceId();
        }
    }

    @Override
    public void destroy() {
        log.info("Destroy Trace filter......");
    }
}

```



## 拦截器拦截地址

```java
package com.lee.traceiddemo.config;

import com.lee.traceiddemo.filter.TraceFilter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;

@Slf4j
@Configuration
public class WebConfiguration {
    @Bean
    @ConditionalOnMissingBean({TraceFilter.class})
    @Order(Ordered.HIGHEST_PRECEDENCE + 101)
    public FilterRegistrationBean<TraceFilter> traceFilterBean() {
        FilterRegistrationBean<TraceFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new TraceFilter());
        bean.addUrlPatterns("/*");
        return bean;
    }
}

```



## 日志配置 

日志地址：`classpath:/resources/logback.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="10 seconds">
    <!--property 的name是变量的名称，value是变量的值-->
    <!-- 日志存放路径 -->
    <property name="log.path" value="logs" />
    <!-- 日志输出格式 -->
    <!--
        %d{HH:mm:ss.SSS}: 表示日志生成的时间戳，精确到毫秒，格式为小时：分钟：秒：毫秒。
        [ %thread ]: 输出产生日志事件的线程名，括号中的百分号不是必需的，但通常用来美化输出使其易于阅读。
        %-5level: 日志级别，左对齐并至少占据5个字符宽度，比如DEBUG、INFO等，如果级别名称短于5个字符，则用空格填充。
        %logger{20}: 输出Logger的全名，但是只显示前20个字符，如果名字超过20个字符，则会被截断并在末尾添加省略号(...)。
        [%method,%line]: 输出产生日志事件的方法名和行号，这里可能需要自定义解析器或额外配置才能正确提取方法名和行号，标准的PatternLayout不支持这样的组合形式，一般会分开写成%method和%line。
        [%X{TraceId}]: 输出MDC (Mapped Diagnostic Context) 中键为TraceId的值，MDC是一种可以在多线程环境中关联特定上下文数据的方式，这里的TraceId通常用于跟踪分布式系统中的请求链路。
        %msg%n: 输出日志消息内容，并在消息后添加一个换行符(\n)。
    -->
    <property name="log.pattern" value="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{20} - [%method,%line] - [%X{TraceId}] - %msg%n" />

    <!-- 控制台输出 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!--显示日志最低级别，默认是info-->
        <!--         <filter class="ch.qos.logback.classic.filter.ThresholdFilter">-->
        <!--             <level>debug</level>-->
        <!--         </filter>-->

        <encoder>
            <pattern>${log.pattern}</pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 系统info日志输出 -->
    <appender name="file_info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/info/sys-info.log</file>
        <!-- 基于时间创建日志文件(最常见，建议使用) -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件名格式 -->
            <fileNamePattern>${log.path}/sys-info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志最大的历史 60天 -->
            <maxHistory>60</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>

        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的级别 -->
            <level>INFO</level>
            <!-- 匹配时的操作：接收（记录） -->
            <onMatch>ACCEPT</onMatch>
            <!-- 不匹配时的操作：拒绝（不记录） -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 系统error日志输出 -->
    <appender name="file_error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/error/sys-error.log</file>
        <!-- 循环政策：基于时间创建日志文件 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志文件名格式 -->
            <fileNamePattern>${log.path}/sys-error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志最大的历史 60天 -->
            <maxHistory>60</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>

        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 过滤的级别 -->
            <level>ERROR</level>
            <!-- 匹配时的操作：接收（记录） -->
            <onMatch>ACCEPT</onMatch>
            <!-- 不匹配时的操作：拒绝（不记录） -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="async_file_info" class="ch.qos.logback.classic.AsyncAppender">
        <!--  当队列的剩余容量小于这个阈值并且当前日志level为 TRACE, DEBUG or INFO ，则丢弃这些日志。  -->
        <discardingThreshold>0</discardingThreshold>
        <!--  更改默认的队列的深度,该值会影响性能.默认值为256  -->
        <queueSize>1024</queueSize>
        <!--  新增这行为了打印栈堆信息  -->
        <includeCallerData>true</includeCallerData>
        <!--  添加附加的appender,最多只能添加一个  -->
        <appender-ref ref="file_info"/>
    </appender>
    <appender name="async_file_error" class="ch.qos.logback.classic.AsyncAppender">
        <!--  当队列的剩余容量小于这个阈值并且当前日志level为 TRACE, DEBUG or INFO ，则丢弃这些日志。  -->
        <discardingThreshold>0</discardingThreshold>
        <!--  更改默认的队列的深度,该值会影响性能.默认值为256  -->
        <queueSize>1024</queueSize>
        <!--  新增这行为了打印栈堆信息  -->
        <includeCallerData>true</includeCallerData>
        <!--  添加附加的appender,最多只能添加一个  -->
        <appender-ref ref="file_error"/>
    </appender>
    <appender name="async_file_debug" class="ch.qos.logback.classic.AsyncAppender">
        <!--  当队列的剩余容量小于这个阈值并且当前日志level为 TRACE, DEBUG or INFO ，则丢弃这些日志。  -->
        <discardingThreshold>0</discardingThreshold>
        <!--  更改默认的队列的深度,该值会影响性能.默认值为256  -->
        <queueSize>1024</queueSize>
        <!--  新增这行为了打印栈堆信息  -->
        <includeCallerData>true</includeCallerData>
        <!--  添加附加的appender,最多只能添加一个  -->
        <appender-ref ref="file_debug"/>
    </appender>

    <!-- 系统模块日志级别控制  -->
    <logger name="cn.xj" level="info" />
    <!-- Spring日志级别控制  -->
    <logger name="org.springframework" level="warn" />

    <!--系统操作日志-->
    <root level="info">
        <appender-ref ref="console" />
        <appender-ref ref="async_file_info" />
        <appender-ref ref="async_file_error" />
    </root>
</configuration>
```



## traceId实战

如果已知traceId，查询上下文信息：`grep -C5 27452AA6FCB247B18AE05EA51F587ED6 sys-info.log`。`-C5`表示携带上下5行日志，`sys-info.log`是日志文件。
