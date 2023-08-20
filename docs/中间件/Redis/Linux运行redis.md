# Linux 运行 Redis

[toc]



## Linux中安装Redis

方法一：下载源码并编译。

1. 在Linux中使用`wget`等工具去官网下载Redis的Linux版本。

2. 下载成功后，将解压包放到`/opt`目录。

3. 安装`gcc`（百度搜索指令），安装成功后查看版本。

4. 解压Redis

   ```
   tar -zxvf redis-7.0.4.tar.gz
   ```

5. 进入解压好的redis文件，执行make命令进行编译。

   ```
   cd redis-7.0.4
   make
   ```

   如果出现错误，清空并重装redis。

   ```
   make disclean
   ```

6. 安装。

   ```
   make insall
   ```



方法二：CentOS 7 中安装Redis

1. `yum update`。
2. `yum install redis`。
3. 启动Redis服务：`systemctl start redis`。
4. 检查Redis是否运行：`systemctl status redis`。
5. 输入：`redis-cli ping`，如果返回`PONG`，证明Redis正常工作。

> 此时我想设置Redis的密码，应该这样做：

6. 打开 Redis 配置文件 `/etc/redis/redis.conf`。

7. 查找并取消注释 `requirepass` 选项，并将密码设置为`123456`：`requirepass 123456`。

8. 重启 Redis 服务：`sudo systemctl restart redis`。

9. 验证Redis服务是否正常运行：使用`redis-cli`启动Redis客户端程序并尝试连接到Redis服务器，如果连接成功，会显示以下内容：

   ```bash
   127.0.0.1:6379>
   ```

10. 虽然我们启动Redis客户端并成功连接上Redis服务器，但是我们还没有验证身份。输入`ping`，发现当前没有权限访问。

11. Redis客户端命令行中输入`AUTH 123456`，返回`OK`，此时获得权限。再输入`ping`，即可返回`PONG`。

> 到此为止，可以在本地访问Redis服务。然而在其他主机上还是不能访问Redis服务，这是因为 Redis 的默认配置只允许本地连接。要允许其他主机连接 Redis，您需要更新 Redis 的配置文件。

12. 打开 Redis 配置文件 `/etc/redis/redis.conf`。
13. 在文件中查找 `bind 127.0.0.1`，并将其注释掉或者将其改为 `bind 0.0.0.0`。这将允许 Redis 接受来自任何 IP 地址的连接。
14. 重启 Redis 服务：`systemctl restart redis`。
15. 在其他主机上输入：`redis-cli -h <redis服务器的ip地址>` 即可远程访问Redis服务。
16. 结束Redis服务端：`systemctl stop redis`。



方法三：Ubuntu 22.04 中安装Redis

1. 更新 apt 索引：`sudo apt update`。
2. 安装Redis：`sudo apt install redis-server`。
3. 之后的步骤和上述 CentOS 7 一致。



## 启动redis

方法一：前台启动。（不推荐）

```
redis-server
```

方法二：后台启动：

1. 将`redis.conf`拷贝到`/etc/`下，将`daemonize no`改成`daemonize yes`

2. 启动redis，`redis-server /etc/redis.conf`

3. 用客户端访问：`redis-cli`

4. 测试验证： `ping`

5. Redis关闭
   单实例关闭：`redis-cli shutdown`
   也可以进入终端后再关闭：`shutdown`

6. 建议给redis设置密码：

   1. 设置密码

      ```
      config set requirepass 123456
      ```

   2. 查询密码

      ```
      config get requirepass
      ```

   3. 登录时输入密码

      ```
      auth 123456
      ```

