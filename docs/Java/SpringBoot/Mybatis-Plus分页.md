# Mybatis-Plus分页

[toc]

## 前置工作

1. 基于springboot2.77。

2. 能够正常运行MySQL和Mybatis-Plus。



## 实战代码

### 配置代码

只需要直接粘贴配置代码即可。

```java
package com.lee.mybatisplugin.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author 晓龙coding
 */
@Configuration
public class MybatisPlusConfig {
    /**
     * 配置分页功能的拦截器对象
     * @return
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```



### 测试代码

```java
package com.lee.mybatisplugin;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.lee.mybatisplugin.mapper.UserMapper;
import com.lee.mybatisplugin.pojo.User;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;
import java.util.List;

@SpringBootTest
class MybatisPluginApplicationTests {
    @Resource
    private UserMapper userMapper;
    @Test
    void test() {
        Page<User> page = new Page<>(1, 10);  // 查询第1页，每页10条数据
        LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
        // 一定不要查询总数据数，否则数据量大的时候，查询效率很低
        Page<User> userPage = userService.page(page, queryWrapper, false);
        List<User> records = userPage.getRecords();
        for (User record : records) {
            System.out.println(record);
        }
    }
}
```

