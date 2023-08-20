# 搭建Docker环境

[toc]

## 安装Docker CE

Docker有两个分支版本：Docker CE和Docker EE，即社区版和企业版。本教程基于CentOS 7安装Docker CE。

1. 执行如下命令，安装Docker的依赖库。

   `yum install -y yum-utils device-mapper-persistent-data lvm2`

2. 执行如下命令，添加Docker CE的软件源信息。

   `yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`

3. 执行如下命令，安装Docker CE。

   `yum makecache fast`

   `yum -y install docker-ce`

4. 执行如下命令，启动Docker服务。

   `systemctl start docker`



## 配置阿里云镜像仓库（镜像加速）

Docker的默认官方远程仓库是hub.docker.com，由于网络原因，下载一个Docker官方镜像可能会需要很长的时间，甚至下载失败。为此，阿里云容器镜像服务ACR提供了官方的镜像站点，从而加速官方镜像的下载。下面介绍如何使用阿里云镜像仓库。

1. 复制容器镜像服务控制台地址，在FireFox浏览器打开新页签，粘贴并访问云容器镜像服务控制台。网址：`https://cr.console.aliyun.com/`

2. 在容器镜像服务控制台左侧导航栏中，选择**镜像工具>镜像加速器**。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/3dc6daf967da4e518808f43d7b93897f.png)

3. 在**镜像加速器**页面的加速器区域，单击**复制**。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/826b50ddb0cc4bd1a8c4fe3c9b4459c0.png)

4. 切换至终端页面。执行如下命令配置Docker的自定义镜像仓库地址。请将下面命令中的镜像仓库地址 https://kqh8****.mirror.aliyuncs.com 替换为上一步阿里云为您提供的专属镜像加速地址。最后在终端输入：

   ```
   tee /etc/docker/daemon.json <<-'EOF' {  "registry-mirrors": ["https://kqh8****.mirror.aliyuncs.com"] } EOF
   ```

5. 重新加载服务配置文件。

   `systemctl daemon-reload`

6. 重启Docker服务。

   `systemctl restart docker`



## 使用Docker安装Nginx服务

1. 查看Docker镜像仓库中Nginx的可用版本。

   `docker search nginx`

   命令返回如下页面。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB1zTRaHRr0gK0jSZFnXXbRRXXa-1090-592.png)

2. 拉取最新版的Nginx镜像。

   `docker pull nginx:latest`

   命令返回如下页面。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB1zcA4HpP7gK0jSZFjXXc5aXXa-688-195.png)



3. 查看本地镜像。

   `docker images`

   命令返回如下页面。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB1igZ2HpY7gK0jSZKzXXaikpXa-731-88.png)



4. 运行容器。

   `docker run --name nginx-test -p 8080:80 -d nginx`

​	命令参数说明：

- --name nginx-test：容器名称。

- -p 8080:80： 端口进行映射，将本地8080端口映射到容器内部的80端口。

- -d nginx： 设置容器在后台一直运行。

  ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/TB1CDRaHRr0gK0jSZFnXXbRRXXa-560-69.png)



5. 在**Chromium浏览器**打开新页签，在地址栏输入http://服务器公网ip:8080访问Nginx服务。

   ![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/8019013f0db24c889c0c37e3b7773b9a.png)