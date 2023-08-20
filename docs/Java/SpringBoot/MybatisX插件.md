# Mybatis插件

[toc]

## 参考文档

[详细文档](https://www.cnblogs.com/6543x1/p/15484098.html)



## Mybatis插件能做什么

可以通过已经创建好的数据库，快速生成Mapper、Service和实体类的代码。



## 如何使用

1. 基于idea，在插件市场中安装MybatisX。

2. 通过idea自带的连接数据库的工具，连接上自己的数据库。

3. 选中指定的数据表，右键，点击【MybatisX-Generator】。

4. `class name strategy`选择`camel`，因为我习惯将类名设置为驼峰格式。

   点击下一步。

5. `annotation`选项选择`Mybatis-Plus 3`，`options`多选项除了不选`Actual Column`，其余都选上。

6. `template`选项选择`mybatis-plus3`。

7. 点击Finish，则自动生成代码。