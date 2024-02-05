# Mybatis-Plus的CRUD



## 前言

mp中，可以用service对象和mapper对象做crud，通常来说，service对象的功能更强。



## 实战

> 数据表结构

```SQL
create table user
(
    id        int auto_increment
        primary key,
    firstname varchar(20) null,
    lastname  varchar(20) null,
    password  varchar(20) null,
    gender    varchar(2)  null,
    constraint idx_name
        unique (firstname, lastname)
);
```

> CRUD

```Java
package com.lee.mybatisplusdemo;

import cn.hutool.core.thread.ThreadUtil;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.LambdaUpdateWrapper;
import com.lee.mybatisplusdemo.mapper.UserMapper;
import com.lee.mybatisplusdemo.pojo.User;
import com.lee.mybatisplusdemo.service.UserService;
import jakarta.annotation.Resource;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.concurrent.CountDownLatch;

@SpringBootTest
class MybatisPlusDemoApplicationTests {
    @Resource
    private UserService userService;

    @Resource
    private UserMapper userMapper;

    private User getUser() {
        return User.builder().firstname("晓").lastname("龙").gender("男").password("123").build();
    }

    private List<User> getUserList(int count) {
        List<User> userList = new ArrayList<>();
        for (int i = 1; i <= count; i++) {
            User user = User.builder().firstname("firstname-" + i).lastname("lastname-" + i).gender(i % 2 == 0 ? "女" : "男").password("pwd" + i).build();
            userList.add(user);
        }
        return userList;
    }

    /**
     * 最简单的，创建一个用户
     */
    @Test
    void createUser() {
        User user = getUser();

        // service
        // 如果因为unique key冲突等原因，通常会报错，而不是success为false
        boolean success = userService.save(user);
        if (success) {
            System.out.println("创建成功");
        } else {
            System.out.println("创建失败");
        }

        // mapper
        // 如果因为unique key冲突等原因，通常会报错，而不是count=0
        int count = userMapper.insert(user);
        System.out.println(count);
    }

    /**
     * 查询firstname和lastname是否存在，若存在，则更新除了id的所有字段
     */
    @Test
    void updateUser() {
        User user = getUser();
        user.setGender("女");
        user.setPassword("666");
        // service
        LambdaUpdateWrapper<User> userLambdaUpdateWrapper = new LambdaUpdateWrapper<>();
        userLambdaUpdateWrapper
                .set(User::getPassword, user.getPassword()).set(User::getGender, user.getGender())
                .eq(User::getFirstname, user.getFirstname()).eq(User::getLastname, user.getLastname());
        boolean success = userService.update(userLambdaUpdateWrapper);
        if (success) {
            System.out.println("update success");
        } else {
            System.out.println("update fail");
        }

        // mapper
//        LambdaUpdateWrapper<User> userLambdaUpdateWrapper = new LambdaUpdateWrapper<>();
//        userLambdaUpdateWrapper
//                .set(User::getPassword, user.getPassword()).set(User::getGender, user.getGender())
//                .eq(User::getFirstname, user.getFirstname()).eq(User::getLastname, user.getLastname());
//        int affectedRowCount = userMapper.update(userLambdaUpdateWrapper);
//        System.out.println(affectedRowCount);
    }

    @Test
    void batchCreateUser() {
        List<User> userList = getUserList(3);

        // service
        // 要么全部创建成功，要么全部创建失败
        boolean success = userService.saveBatch(userList);
        if (success) {
            System.out.println("批量创建成功");
        } else {
            System.out.println("批量创建失败");
        }

        // mapper 没有批量创建操作
    }

    /**
     * 插入or更新用户
     * 案例：如果firstname存在，则更新；否则插入。
     * 建议分成2步，先查询，在判断是更新还是插入。不要用官方提供的saveOrUpdate
     */
    @Test
    void upsertUser() {
        User user = getUser();
        LambdaQueryWrapper<User> userLambdaQueryWrapper = new LambdaQueryWrapper<>();
        userLambdaQueryWrapper.eq(User::getFirstname, "firstname-11").last(" limit 1");
        // 如果有unique索引保证数据唯一，也可以用selectOne
        List<User> userList = userMapper.selectList(userLambdaQueryWrapper);
        if (userList == null) {
            userMapper.insert(user);
        } else {
            for (User u : userList) {
                // 赋值主键，需要确保id是主键且id上有@TableId
                user.setId(u.getId());
                // 根据主键更新数据
                userMapper.updateById(user);
            }
        }
    }


    /**
     * 批量创建部分用户
     * 案例：我有一些用户数据，更新数据库中不存在的数据。判断两个数据相同的标准是:firstname相同且lastname相同
     */
    @Test
    void batchUpdateUser() {
        // 初始化数据
        List<User> userList = getUserList(1000);

        // service

        // 查询满足条件的用户
        LambdaQueryWrapper<User> userLambdaQueryWrapper = new LambdaQueryWrapper<>();
        for (User user : userList) {
            // or (firstname = ? and lastname = ?)
            userLambdaQueryWrapper.or(i -> i.eq(User::getFirstname, user.getFirstname()).eq(User::getLastname, user.getLastname()));
        }
        List<User> oldUserList = userService.list(userLambdaQueryWrapper);
        // 筛选出需要添加的用户
        List<User> toInsertUserList = new ArrayList<>();
        for (User user : userList) {  // 这里为了操作简单，采用两层for循环。
            boolean find = false;
            for (User oldUser : oldUserList) {
                if (Objects.equals(user.getFirstname(), oldUser.getFirstname())
                && Objects.equals(user.getLastname(), oldUser.getLastname())) {
                    find = true;
                    break;
                }
            }
            if (! find) {
                toInsertUserList.add(user);
            }
        }
        boolean ok = userService.saveBatch(toInsertUserList);
        if (ok) {
            System.out.println("save ok");
        } else {
            System.out.println("wrong~");
        }

        // mapper 由于 mapper 没有批量插入的功能，所以不建议用mapper层的接口
    }

    /**
     * 所有的批量操作都可以考虑用线程池操作
     * @throws InterruptedException
     */
    @Test
    void batchUpdateUserMultiThread() throws InterruptedException {
        List<User> userList = new ArrayList<>();
        for (int i = 1; i <= 100; i++) {
            User user = new User();
            user.setFirstname("姓" + i);
            user.setLastname("名" + i);
            user.setPassword("" + i);
            if (i % 2 == 0) {
                user.setGender("女");
            } else {
                user.setGender("男");
            }
            userList.add(user);
        }

        long t1 = System.nanoTime();
        CountDownLatch countDownLatch = ThreadUtil.newCountDownLatch(userList.size());
        for (User user : userList) {
            ThreadUtil.execute(() -> {
                boolean ok = userService.save(user);
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        long t2 = System.nanoTime();
        System.out.println("执行时间:" + (t2 - t1) / 1_000_000 + "毫秒");
    }
}
```
