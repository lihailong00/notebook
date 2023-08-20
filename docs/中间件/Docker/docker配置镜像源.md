# docker配置镜像源

[toc]

## 安装docker软件时配置镜像源

安装docker时，按照[官方文档](https://docs.docker.com/compose/install/other/#on-linux)的步骤即可。由于docker的安装资源文件在国外，下载速度较慢，可以找一个[国内镜像源](https://get.daocloud.io/#install-compose)。不过根据个人经验，没有太大必要使用国内镜像源。

### 使用镜像源安装docker

1. [参考文档](https://get.daocloud.io/#install-compose)。如果使用国内镜像源，会输入这样的指令：`curl -sSL https://get.daocloud.io/docker | sh`。大致意思是从 DaoCloud 服务器上下载 Docker 安装脚本，然后使用 sh 命令解释器运行该脚本，以完成 Docker 的安装。

2. 运行docker：`systemctl start docker`。



## 从docker hub上pull软件时配置镜像源

由于docker hub在国外，下载速度很慢，所以强烈建议配置镜像源。具体步骤如下：

1. 编辑 `/etc/docker/daemon.json` 配置文件：（关于docker hub镜像源地址，可以在网上搜）

   ```
   {
     "registry-mirrors": [
       "https://ustc-edu-cn.mirror.aliyuncs.com",
       "https://hub-mirror.c.163.com",
       "https://mirror.baidubce.com"
     ]
   }
   ```

2. 重新加载守护程序的配置文件：`systemctl daemon-reload `。

3. 重启docker：`systemctl restart docker`。

4. 检查配置是否生效：`sudo docker info`。如果看到类似下面的内容，则配置成功：

   ```
   Registry Mirrors:
     https://ustc-edu-cn.mirror.aliyuncs.com/
     https://hub-mirror.c.163.com/
     https://mirror.baidubce.com/
   ```