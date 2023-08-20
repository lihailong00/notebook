# mysql的安装和删除

[toc]

## Windows安装mysql





## Windows删除mysql



![QQ截图20220627123436](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/QQ%E6%88%AA%E5%9B%BE20220627123436.png)

以**管理员**的身份运行命令行，然后输入`sc delete MySQL80`。（注意：MySQL80是我的服务名称，你的电脑可能不是这个名字）

## Linux(CentOS7)安装mysql

1. 去[mysql Yum Repository](https://dev.mysql.com/downloads/repo/yum/)选择适合自己版本的mysql的RPM包。

   ![QQ截图20220627133042](QQ截图20220627133042.png)

   ![2022-06-27_133135](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/2022-06-27_133135.png)



2. 在linux命令行中使用wget下载该rpm包。输入：`wget 链接地址`即可下载mysql的rpm包。

3. 安装mysql的rpm包：

   ```
   sudo rpm -Uvh platform-and-version-specific-package-name.rpm
   ```

   其中platform-and-version-specific-package-name.rpm是你下载的rpm包的名称。
   我的包名是：mysql80-community-release-el7-6.noarch.rpm
   所以我输入：

   ```
   sudo rpm -Uvh mysql80-community-release-el7-6.noarch.rpm
   ```

   备注：`yum update`会更新我的mysql的rpm包

4. 如果想安装mysql5.7，进入`/etc/yum.repos.d/mysql-community.repo`：

   ```
   [mysql57-community]
   ...
   enabled=1
   ...
   
   [mysql80-community]
   ...
   enable=0
   ...
   ```

   对目标版本设置`enable=1`，对其他版本设置`enable=0`。

5. 安装mysql

```
sudo yum install mysql-community-server
```





## Linux(CentOS7)删除mysql

1. 假定linux已安装mysql，输入`service mysqld status`查看当前mysql数据库的状态，并输入`service mysqld stop`关闭mysql。

2. 输入`rpm -ga|grep -i mysql`查看当前安装了哪些mysql组件。

   ![QQ截图20220627130807](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/QQ%E6%88%AA%E5%9B%BE20220627130807.png)

3. 依次删除相关组件。

   ```
   rpm -ev --nodeps mysql-community-client-plugins-8.0.29-1.el7.x86_64
   rpm -ev --nodeps mysql-community-libs-8.0.29-1.el7.x86_64
   rpm -ev --nodeps mysql80-community-release-el7-6.noarch
   rpm -ev --nodeps mysql-community-client-8.0.29-1.el7.x86_64
   rpm -ev --nodeps mysql-community-common-8.0.29-1.el7.x86_64
   rpm -ev --nodeps mysql-community-icu-data-files-8.0.29-1.el7.x86_64
   rpm -ev --nodeps mysql-community-server-8.0.29-1.el7.x86_64
   ```

4. 查看与mysql相关的残留文件。

   输入：`find / -name mysql`

​	删除以下目录中的mysql残留文件：

```
/var/spool/*
/var/lib/mysql
/usr/*
/run
```

