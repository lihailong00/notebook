# 基于阿里云ECS服务器自制云盘

[toc]

## 1.安装OwnCloud

OwnCloud是一款开源的云存储软件，基于PHP的自建网盘。基本上是私人使用，没有用户注册功能，但是有用户添加功能，你可以无限制地添加用户，OwnCloud支持多个平台（windows，MAC，Android，IOS，Linux）。

1. 通过terminal连接ECS服务器。执行以下命令，添加一个新的软件源。

   ```
   cd /etc/yum.repos.d/
   wget --no-check-certificate https://download.opensuse.org/repositories/isv:ownCloud:server:10/CentOS_7/isv:ownCloud:server:10.repo
   ```

2. 执行以下命令进

   ```
   cd /root/
   ```

3. 执行以下命令安装OwnCloud-files。

   ```
   yum -y install https://labfileapp.oss-cn-hangzhou.aliyuncs.com/owncloud-complete-files-10.5.0-3.1.noarch.rpm
   ```

4. 执行以下命令查看安装是否成功。

   ```
   ll /var/www/html
   ```

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB15MD6HxD1gK0jSZFyXXciOVXa-629-78.png)



## 2. 安装Apache服务

1. 执行以下命令安装Apache服务。

   ```
   yum install httpd -y
   ```

2.  执行以下命令启动Apache服务。

   ```
   systemctl start httpd.service
   ```

3. 打开浏览器输入体验平台创建的ECS的弹性公网IP。如果出现如下图内容表示Apache安装成功。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB1vopec5cKOu4jSZKbXXc19XXa-1652-418.png)



4. 添加OwnCloud配置：

   1.  执行以下命令打开Apache配置文件。

      ```
      vim /etc/httpd/conf/httpd.conf
      ```

   2.  按i键进入文件编辑模式，然后在<Directory>内容后添加以下内容。

      ```
      # owncloud config
      Alias /owncloud "/var/www/html/owncloud/"
      <Directory /var/www/html/owncloud/>
          Options +FollowSymlinks
          AllowOverride All
          <IfModule mod_dav.c>
              Dav off
          </IfModule>
          SetEnv HOME /var/www/html/owncloud
          SetEnv HTTP_HOME /var/www/html/owncloud
      </Directory>
      ```

   3.  按esc键退出编辑模式，然后输入:wq保存并退出配置文件。



## 3. 安装并配置PHP

由于OwnCloud是基于PHP开发的云存储软件，需要PHP运行环境，请根据以下步骤完成OwnCloud工作环境的配置。

1.  执行以下命令手动更新rpm源。

   ```
   rpm -Uvh https://labfileapp.oss-cn-hangzhou.aliyuncs.com/epel-release-latest-7.noarch.rpm 
   rpm -Uvh https://labfileapp.oss-cn-hangzhou.aliyuncs.com/webtatic-release.rpm   
   ```

2.  执行以下命令安装PHP 7.2版本。

   说明：OwnCloud只支持PHP 5.6+。

   ```
   yum -y install php72w
   yum -y install php72w-cli php72w-common php72w-devel php72w-mysql php72w-xml php72w-odbc php72w-gd php72w-intl php72w-mbstring
   ```

3.  执行以下命令检测PHP是否安装成功。

   ```
   php -v
   ```

4.  将PHP配置到Apache中：

   1. 执行以下命令，找到php.ini文件目录。

      ```
      find / -name php.ini
      ```

   2. 执行以下命令打开httpd.conf文件。

      ```
      vi /etc/httpd/conf/httpd.conf
      ```

   3.  按i键进入文件编辑模式，然后在文件最后添加以下内容。

      ```
      PHPIniDir /etc/php.ini
      ```

   4.  按esc键退出编辑模式，然后输入:wq保存并退出配置文件。

   5.  执行以下命令，重启Apache服务。

      ```
      systemctl restart httpd.service
      ```



## 4. 配置OwnCloud

完成上述配置后，您就可以登录OwnCloud创建个人网盘了。

1. 在自己的浏览器中，输入ECS弹性IP/owncloud，例如1.1.1.1/owncloud。

2. 自定义输入管理员账号和密码，然后单击**存储&数据库**，选择SQLite，最后单击**安装完成**。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/bf9db4c580a34f298210df3f448b1c09.png)

3. 输入已创建的用户名和密码登录Owncloud。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB11XYhxHr1gK0jSZR0XXbP8XXa-1005-477.png)

​	登录成功界面如下：

​	![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB117rPzQY2gK0jSZFgXXc5OFXa-1896-622.jpg)



## 5. 挂载NAS服务

完成OwnCloud初始化之后就可以将NAS存储包挂载到您的网盘服务器上。

1. 打开**无痕浏览器**页面，复制 **云产品资源** 列表中 「**一键复制子账号登陆链接**」 粘贴到浏览器，输入 **云产品资源列表** 中的**子账号密码**，即可登录当前子账号。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/d48cbb9447564e9abfceed05be19bfed.png)



2. 输入云产品资源提供的子用户名和密码，登录阿里云控制台。在产品列表页，搜索NAS，然后单击**文件存储NAS**。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB1IhzZHqL7gK0jSZFBXXXZZpXa-617-313.png)

**说明：**如果出现RAM子账号没有权限的提示，请您关闭提示即可，不会影响体验操作。



3. 在左侧导航栏中，单击**文件系统列表**。在页面上方，选择资源提供的地域 ，例如：**华东2 上海**。在文件系统列表页面，可以根据云产品资源列表中提供的**nasFileSystemId**找到您的NAS资源，然后单击**文件系统 ID**，进入文件系统详情页。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/9425cdd5eb9a4db2a94fe3d5d4e1c9d2.png)



4. 选择**挂载使用**，然后单击**添加挂载点**，专有网络选择**cn-shanghai-vpc-csn**、交换机选择**cn-shanghai-b-csn**和权限组选择**VPC默认权限组**，最后单击**确定**。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB1IZrLzND1gK0jSZFyXXciOVXa-1664-682.jpg)



5.  切换至**命令行终端页面**，执行以下命令安装**cifs-utils工具包**。

   ```
   sudo yum -y install cifs-utils
   ```

6.  执行以下命令，查看apache的uid和gid。

   ```
   cat /etc/passwd|grep apache
   ```

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/22381a0bbeae4d32987f659bd83ee944.png)



7.  切换至无痕浏览器，在文件存储NAS控制台页签，单击通过命令行挂载到ECS，挂载文件系统选择Linux，在Linux ECS上安装CIFS客户端选择Centos，查看挂载SMB文件系统命令。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/144716207817470da944d28b72a4cc30.png)



8. 复制挂载SMB文件系统的命令，并且您需要将命令中/mnt改为/var/www/html/owncloud/data/admin/files，uid和gid值改为apache的uid和gid。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/87c43b98071e444aa0ec70e9ab6c0c28.png)



9. 切换至命令行终端页面，执行上一步骤的修改后的挂载命令。

10. 执行以下命令查看挂载是否成功。

    ```
    df -h | grep aliyun
    ```

    ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/af1057dd20104b15bc8cfa432490e02c.jpeg)



11. 您现在可以在OwnCloud网盘中，新建文件夹并上传文件，并且可以在/var/www/html/owncloud/data/admin/files目录下查找到您上传的文件。

    ```
    cd /var/www/html/owncloud/data/admin/files
    ```

    