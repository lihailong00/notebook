# springboot生成ID

[toc]



## 雪花算法

前提：spring boot集成mybatis-plus。可以参考我写的《springboot集成mybatis-plus操作MySQL》这篇博客。

在实体类的ID属性上添加注解`@TableId(value = "id")`即可：

```java
@TableId(value = "id")
private Long id;
```



## 精度丢失问题

雪花算法生成的ID传到前端会导致精度丢失，建议使用`jackson`序列化工具，并添加如下配置即可：

```java
package com.lee.medicine.config;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;

@Configuration
public class JacksonConfig {
    @Bean
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
      ObjectMapper objectMapper = builder.createXmlMapper(false).build();
      // 全局配置序列化返回 JSON 处理
      SimpleModule simpleModule = new SimpleModule();
      //JSON Long ==> String
      simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
      objectMapper.registerModule(simpleModule);
      return objectMapper;
    }
}
```

