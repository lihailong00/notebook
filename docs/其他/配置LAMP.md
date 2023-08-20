# 基于CentOS的服务器上配置LAMP

[toc]

## 安装Apache服务

Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。

1. 执行如下命令，安装Apache服务及其扩展包。

   ```
   yum -y install httpd httpd-manual mod_ssl mod_perl mod_auth_mysql
   ```

   备注：如果提示错误信息`Linux yum安装httpd报错 No package httpd available ...`，执行以下命令：

   ```
   yum --disableexcludes=all install -y httpd
   ```

   

2. 执行如下命令，启动Apache服务。

   `systemctl start httpd.service`

3. 在本地电脑的浏览器的址栏中，输入**服务器公网IP地址**，并按Enter键。

   若返回页面如下图所示，说明Apache服务启动成功。

![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/39b506798c2344948ec193656805fcd1.jpeg)



## 安装并配置MySQL

MySQL是最流行的关系型数据库管理系统，在WEB应用方面MySQL是最好的 RDBMS(Relational Database Management System：关系数据库管理系统)应用软件之一。

1. 执行以下命令，下载并安装MySQL官方的`Yum Repository`。

   ```
   wget http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
   ```

   ```
   yum -y install mysql57-community-release-el7-10.noarch.rpm
   ```

   ```
   yum -y install mysql-community-server --nogpgcheck
   ```

   **如何选择其他版本的mysql？**

   进入mysql官网的[yum repository](https://dev.mysql.com/downloads/repo/yum/)，选择相应的版本即可。

   ![QQ截图20220626225514](QQ截图20220626225514.png)

2. 运行以下命令查看MySQL版本号。

   `mysql -V`

3. 执行以下命令，启动 MySQL 数据库。

   `systemctl start mysqld.service`

4. 执行以下命令，查看MySQL初始密码。

   `grep "password" /var/log/mysqld.log`

   或者`vim /var/log/mysqld.log` 查找相关密码。

5. 执行以下命令，登录数据库。

   `mysql -u root -p`

6. 执行以下命令，修改MySQL默认密码。

   `set global validate_password_policy=0;  #修改密码安全策略为低（只校验密码长度，至少8位）。`

   `ALTER USER 'root'@'localhost' IDENTIFIED BY '12345678';`

7. 执行以下命令，授予root用户远程管理权限。

   `GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '12345678';`（弃用）

   ```
   grant all privileges on *.* to root@'localhost';
   ```

   

8. 输入`exit`退出数据库。



## 安装PHP

PHP（PHP：Hypertext Preprocessor递归缩写）中文名字是：“超文本预处理器”，是一种广泛使用的通用开源脚本语言，适合于Web网站开发，它可以嵌入HTML中。编程范型是面向对象、命令式编程的。

1. 执行以下下命令，安装PHP环境。**（又不行了）**

   `yum -y install php php-mysql gd php-gd gd-devel php-xml php-common php-mbstring php-ldap php-pear php-xmlrpc php-imap`

2. 执行以下命令创建PHP测试页面。

   `echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php`

3. 执行以下命令，重启Apache服务。

   `systemctl restart httpd`

4. 在本地浏览器的址栏中，访问`http://服务器公网ip/phpinfo.php`，显示如下页面表示PHP语言环境安装成功。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/O1CN01UkJF1I1VPSIKnFRsU_!!6000000002645-2-tps-472-758.png)



## 安装phpMyAdmin

phpMyAdmin是一个MySQL数据库管理工具，通过Web接口管理数据库方便快捷。

1. 执行以下命令，创建phpMyAdmin数据存放目录。

   `mkdir -p /var/www/html/phpmyadmin`

2. 执行以下命令，下载phpMyAdmin压缩包。

   `wget --no-check-certificate https://files.phpmyadmin.net/phpMyAdmin/4.0.10.20/phpMyAdmin-4.0.10.20-all-languages.zip`

3. 执行以下命令，安装unzip并解压phpMyAdmin压缩包。

   `yum install -y unzip`

   `unzip phpMyAdmin-4.0.10.20-all-languages.zip`

4. 执行以下命令，复制phpMyAdmin文件到数据存放目录。

   `mv phpMyAdmin-4.0.10.20-all-languages/*  /var/www/html/phpmyadmin`

5. 在本地浏览器的址栏中，输入http://实例公网 IP/phpmyadmin，访问phpMyAdmin。

   返回页面如下图所示，说明phpMyAdmin安装成功。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/O1CN01WVgI1m1C6hNDDatqj_!!6000000000032-2-tps-620-570.png)



6. 在**phpMyAdmin登录**页面，依次输入MySQL的用户名和密码，单击**执行**。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/O1CN01lXqT5q1TPnM52zgwD_!!6000000002375-2-tps-556-614.png)

​	返回页面如下图所示，表示MySQL连接成功。

![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/O1CN018cHAcS1OkL2eEhHfT_!!6000000001743-2-tps-1313-776.png)