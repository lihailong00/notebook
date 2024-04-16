# Mysql权限管理



[toc]



## 查看Mysql用户

1. 查看所有的用户的host：`SELECT user, host FROM mysql.user`。其中host=localhost时，表示只有本机才能访问数据库；host=%时，所有外部链接都能访问数据库。

```
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
| root             | %         |
+------------------+-----------+
```



2. 查看当前用户：`SELECT CURRENT_USER()`。

```
+----------------+
| current_user() |
+----------------+
| root@localhost |
+----------------+
```



## 创建Mysql用户

1. 创建用户`lihailong`，密码`123456`，所有IP都可以访问：`CREATE USER 'lihailong'@'%' IDENTIFIED BY '123456'`。
2. 创建用户`xiaolong`，密码`123`，仅本机可以访问：`CREATE USER 'xiaolong'@'localhost' IDENTIFIED BY '123456'`。



## 授权

1. 给host为`localhost`用户`lihailong`授予数据库`testdb`的表`user_table`授予增删改查权限：`GRANT select, update, insert, delete ON testdb.user_table TO 'lihailong'@'localhost'`。
2. 如果要授予全部权限，可以用ALL：`GRANT all privileges ON testdb.user_table TO 'lihailong'@'localhost'`。
3. testdb是数据库，user_table是数据表，通配符是`*`。
4. 如果想修改用户`lihailong`的`host`，执行：`update mysql.user set host = 'localhost' where user = 'lihailong'`。
5. 修改用户`lihailong`的密码：`alter user 'lihailong'@'%' identified by 'a839oihjf@#FA#@A%23a';`





## 删除用户

1. 删除用户`'lihailong'@'%'`：`drop user 'lihailong'@'%'`
