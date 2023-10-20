# docker部署MySQL

[toc]



## 部署MySQL单体应用

1. 搜索远程仓库中关于MySQL的镜像：`docker search mysql`，并拉取镜像`docker pull mysql:latest`。

2. 运行docker关于MySQL的容器，并将配置文件。通常要将容器中的`配置文件`、`日志文件`和`数据`挂载到本机，并且将MySQL的密码设置到环境变量中。

```shell
docker run -id \
-p 3307:3306 \
--name mydb \
-v /mnt/docker/mysql/conf:/erc/mysql/conf.d \
-v /mnt/docker/mysql/logs:/logs \
-v /mnt/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=自定义MySQL密码 \
mysql:latest


```

3. 进入MySQL数据库：`docker exec -it 容器名 mysql -u root -p`，然后输入密码即可。之后就能进入MySQL的命令行中。
4. 输入exit就能退出MySQL命令行和MySQL容器。
5. 该MySQL容器会在后台一直运行，除非`docker stop`或者卸载docker才会消失。

【注】如果在虚拟机中使用docker安装MySQL，那么在本机的Navicat中不能使用`localhost`访问MySQL服务，而是要通过虚拟机的ip（比如`192.168.1.301`）加上端口号访问MySQL服务。



## 部署MySQL集群

本案例部署三个MySQL，实现读写分离。

1. 准备主服务器：

   ```
   docker run -id \
   -p 3308:3306 \
   --name 容器名 \
   -v /mnt/docker/mysql/master/conf:/erc/mysql/conf.d \
   -v /mnt/docker/mysql/master/logs:/logs \
   -v /mnt/docker/mysql/master/data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=自定义MySQL密码 \
   mysql:latest
   ```

2. 配置主服务器的内容：

   编辑`/mnt/docker/mysql/master/conf/my.cnf`。

   ```
   [mysqld]
   # 服务器唯一id，默认值1
   server-id=1
   # 设置日志格式，默认值ROW
   binlog_format=STATEMENT
   # 二进制日志名，默认binlog
   # log-bin-binlog
   # 设置需要复制的数据库，默认复制全部数据库
   # binlog-do-db=mytestdb
   # 设置不需要复制的数据库
   # binlog-ignore-db=mysql
   # binlog-ignore-db=infomation-schema
   ```

3. 重启docker容器：`docker restart 容器名`。

4. 进入MySQL服务器：`docker exec -it 容器名 mysql -u root -p`，并输入密码。

5. `grant replication slave on *.* to 'mysql-slave'@'%';`

