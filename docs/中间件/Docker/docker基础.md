# docker实战教程

[toc]

## 如何高效使用docker

[docker官方文档](https://docs.docker.com/)

1. 关于学习具体的指令
   1. 通常用[ ]表示可写或可不写的内容，例如：`docker start [OPTIONS] CONTAINER [CONTAINER...]`表示OPTIONS对应的值可写可不写，并且容器后面可以接很多容器。
   2. 在shell中也可以快速查看API，假定我们要操作容器，输入`docker container`即可，操作镜像，输入`docker image`即可。这两个API最常使用。
   3. 如果想查看某个API的功能，输入`指令 + --help`即可，例如`docker container export --help`。



## 理论知识

[docker官方文档](https://docs.docker.com/)

问题：docker技术采用了哪些Linux内核的特性？

答案：

当你创建一个容器时，docker会为该容器创建新的namespace。



问题：如何理解docker容器？

答案：

1. 容器是宿主机上的一个进程，它与其他进程隔离。

2. 容器是处于运行状态的镜像。



问题：如何理解docker镜像？

答案：简单来说，docker镜像就是软件包。它为运行的容器提供一个独立的文件系统（叫做联合文件系统），同时它也提供容器运行时需要的所有软件、环境、依赖......



![img](https://ask.qcloudimg.com/http-save/developer-news/h2qjlnki50.png?imageView2/2/w/1620)



问题：docker镜像为什么要采用分层原理？

答案：为了共享资源；假定有若干个容器都是基于某个镜像创建的，那么镜像中存放了这些容器需要的软件。举个例子：比如某个镜像中安装了vim软件，那么由它创建的容器也都可以使用vim软件，从而节约资源。



问题：docker镜像可写吗？docker容器可写吗？

答案：镜像不可写，只可读。容器既可读，又可写。



[参考文档](https://circleci.com/blog/docker-image-vs-container/#:~:text=Although%20Docker%20images%20and%20containers%20have%20a%20similar,necessary%20for%20a%20machine%20to%20run%20that%20application.)

问题：既然docker image是只读的，那么如何制作docker image呢？



问题：创建好docker image后，可以修改image中的部分内容吗？如果不能，该如何修改image中的内容呢？





问题：不同容器中运行的程序会相互影响吗？

答案：启动一个docker，相当去创建一个进程。docker属于进程之间的隔离。



问题：docker和虚拟机的区别？

答案：虚拟机模拟一个新的硬件环境，而docker模拟一个新的操作系统。具体体现如下：

1. 启动速度：启动docker相当于启动一个进程；启动虚拟机相当于启动一个操作系统。所以启动docker更快。
1. 占用资源：docker比虚拟机少很多。
1. 隔离性：docker的隔离性比虚拟机弱。docker实现进程之间的隔离。虚拟机实现操作系统之间的隔离。
1. 安全性：docker的安全性弱。**docker中的root和宿主机的root等同**，一旦在docker容器中获得root权限，那么也就获得了宿主机的root权限。





问题：docker 容器和镜像的区别？

​	答：1. 镜像创建好后不可修改，每次通过镜像创建一个容器时，容器都会包含镜像中的内容。

2. 从镜像分层的角度看，镜像包含了一系列重叠的只读层，该镜像创建的容器会基于只读层，创建读写层，不同的实例共享只读层。
3. 容器是一个运行时环境，而镜像是一直存在的。



[参考资料](https://blog.csdn.net/lu_1110/article/details/106533490)

问题：docker常见的base镜像有哪些？

问题：docker虚拟的kernel一定和宿主机一样吗？如果宿主机的操作系统和docker的操作系统不一样，kernel还是一样的吗？

问题：镜像分层是什么意思，包含了只读层和读写层吗？容器分层又是什么意思，包含了只读层和读写层吗？它俩会共享只读层的文件吗？



其他：

问题：dockerhub是什么？国内有类似dockerhub的仓库吗



## docker有什么用

一次镜像，处处运行。



## docker的基本组成

镜像：类比于类

容器：类比于对象

仓库：存放docker镜像



## docker架构图解

目前我记得也不是很清楚

[![img](https://figurebed-1309161819.cos.ap-nanjing.myqcloud.com/img/th)](https://cn.bing.com/images/search?q=Docker 架构图&FORM=IQFRBA&id=1EB3DE06BDC66FA8800F4E9CA7382F9CA7652E3E)





## docker安装教程

[官方文档centos版](https://docs.docker.com/engine/install/centos/)





## docker镜像加速（阿里云）

问题：什么是docker镜像加速？

答案：如果要拉取docker镜像，我们不使用dockerhub，而是使用国内的某些存放docker镜像的网站。因此我们需要配置docker拉取镜像的目标网站。

[配置](https://cr.console.aliyun.com/cn-wulanchabu/instances/mirrors)



## docker命令

对于操作docker，我们需要明确的是，要操作docker的哪些对象？也即操作docker软件、还是操作docker的image，还是操作docker的container。



### 操作docker软件

1. 查看docker运行情况：`docker info`
2. 启动docker：`systemctl start docker`
3. 停止docker：`systemctl stop docker`
4. 重启docker：`systemctl restart docker`
5. 查看docker状态：`systemctl status docker`
6. 开机启动：`systemctl enable docker`
7. 查看docker概要信息：`docker info`
8. 查看docker总体帮助文档：`docker --help`
9. 查看docker命令帮助文档：`docker 具体命令 --help`
10. 查看docker占用的空间：`docker system df`



### 操作docker镜像

1. 列出本地主机上的镜像：`docker images`。

| REPOSITORY   | TAG                  | IMAGE ID   | CREATED      | SIZE     |
| ------------ | -------------------- | ---------- | ------------ | -------- |
| 镜像的仓库源 | 镜像的标签（版本号） | 镜像唯一ID | 镜像创建时间 | 镜像大小 |



2. 去镜像远程仓库查找某个镜像：`docker ` search 镜像名称

​	[docker search 相关文档](https://blog.csdn.net/lw001x/article/details/107152016)



3. 拉取（下载）某个镜像：`docker pull 镜像名字[:latest]`

​	例如：`docker pull mysql:5.7`



4. 删除某个镜像：`docker rmi 镜像名或镜像ID`

​	i 表示 image



5. tar包导入为docker image：`docker image import `





### 操作docker容器

**创建 docker容器**：`docker run -i -t 镜像名 /bin/bash`

`-i`：保证容器的STDIN开启，持久的标准输入是交互式shell的“半边天”。

`-t`：让docker为新创建的容器分配一个为tty终端。

`/bin/bash`：创建好容器后，首先执行该命令，也即打开一个Bash shell。



> 问题：docker如何定位“镜像名”呢？

> 答案：先查找本地。如果没有再查找远程镜像库。默认在DockerHub中查找，如果修改过docker镜像源，则在修改后的镜像源中查找。如果找到，会先将镜像下载到本地，然后基于镜像创建容器。如果还没找到，则无法创建容器（因为没镜像呀~）。



> 问题：新创建的docker容器都有哪些属性呢？

> 答案：有自己的网络、IP地址、用来和宿主机进行通信的桥接网络接口。





**自定义端口和容器名称**：`docker run -it -p 8888:8080 --name t1 tomcat /bin/bash`



**创建守护式容器**：`docker run -d redis:6.0.8`



**查看正在运行的容器**：`docker ps`

**查看所有容器**：`docker ps -a`

**查看容器中正在做什么**：`docker logs 容器名`

**查看某个容器内部信息**：`docker inspect 容器名`

补：进入`/var/lib/docker`能查看更多关于docker的配置信息。



**启动已经停止的容器**：`docker start 容器名`

**停止正在运行的容器**：`docker stop 容器ID或容器名`



**查看容器日志**：`docker logs 容器名`

**查看容器的进程**：`docker top 容器名`

**向容器中添加额外进程**：`docker exec` （一般不单独执行）



**进入正在运行的容器**：

方法一：`docker exec -it 容器名 /bin/bash`

方法二：`docker attach 容器名`

问题：两者有什么区别？

答案：

	1. 方法一是创建一个新的线程进入容器。
	1. 方法二是直接进入容器。



**退出容器**

方法一：`ctrl + d`

方法二：`ctrl + p + q`

问题：使用`ctrl + d`退出容器和`ctrl + p + q`退出容器的区别？

答案：`ctrl + d`会结束线程并退出容器，`ctrl + p + q`不会结束线程并退出游戏。



**重启容器**：`docker restart 容器名`

注意：容器会检查错误代码，然后决定是否重启，默认为不重启。如果想让容器总是重启，则需要这样做`docker restart --restart=always 容器名`。



**问题：如果修改已经分配好端口号的容器端口？**

 

**从容器中拷贝文件到主机**：`docker cp 容器ID:容器内路径 目的主机路径`

**从主机中拷贝文件到容器**：`docker cp 主机文件地址 容器ID:容器内路径`

**容器导出为tar解压包**：`docker export 容器ID > 文件名.tar`

**tar解压包导入为 *镜像* **：`cat 文件名.tar | docker import - 镜像名:镜像版本号`，其中镜像用户、镜像名和版本号随意指定。

注意：**容器**导出为解压包，解压包再导入为**镜像**！



**将容器导出为镜像/制作镜像**：`docker commit -m "add vim" -a "lhl" 容器ID 镜像名称[:镜像版本号]`



**将自己制作的镜像上传至阿里云**：

首先创建一个[命名空间](https://cr.console.aliyun.com/cn-wulanchabu/instance/namespaces)，基于此命名空间创建一个[镜像仓库](https://cr.console.aliyun.com/cn-wulanchabu/instance/repositories)，然后按照阿里的官方文档上传本地镜像至镜像仓库即可。

[阿里云与docker镜像 官方文档](https://developer.aliyun.com/article/888370)

建议：

1. 命名空间作为一些仓库的集合，推荐将一个公司或组织的仓库集中在一个命名空间下面。

   >以公司名称作为命名空间：aliyun、alibaba
   >
   >以团队、组织作为命名空间：misaka-team

2. 仓库是镜像的集合，建议将**一个应用不同版本**的镜像放置在一个仓库中。建议以**软件包名或应用名作为仓库名称**。

>以软件包命名：例如 centos、jetty
>
>以应用命名：例如 console-web、console-service



构建私有镜像仓库：使用docker registry。（我暂时还用不上）





将当前容器myos导出为mycentos.tar到当前目录：`docker export -o mycentos.tar myos`：导出到当前目录。

`docker import mycentos.tar myosimage:1.0`：mycentos.tar文件必须在当前目录下，myosimage表示该tar包导入为镜像时的REPOSITORY（仓库名），1.0表示该镜像的TAG（版本号）。





## docker 容器数据卷

问题：为什么要学这个东西？

答案：假定我们想随时备份某个容器内部的资料，防止该容器被误删后资料丢失，那么我们需要考虑将docker容器中的数据持久化到宿主机中。此时宿主机中的内容和docker容器中的内容共享同一片存储空间。

`docker run -it --privileged=true -v /宿主机绝对路径:/容器内目录 镜像名`



案例：

`docker run -it --privileged=true -v /tmp:/tmp ubuntu:20.04 /bin/bash`





--privileged=true 让docker容器拥有root权限。



## dockerfile

[官方文档](https://docs.docker.com/engine/reference/builder/)

dockerfile是用来构建docker镜像的**文本文件**。假定我们希望镜像中带有vim、jdk、tomcat和nginx，那么我们可以编写一个dockerfile，运行此dockerfile后会在镜像中安装相关软件。





### RUN

使用方法：`RUN yum -y install vim`。



[基本原则的参考文档](https://blog.csdn.net/liumiaocn/article/details/103175638)

基本原则1：尽量减少一个Dockerfile中的RUN命令的个数

> RUN命令在构建时会创建一个新层，如非特殊的需要，建议一个Dockerfile在需要使用RUN命令的时候尽可能的只用一个RUN命令，将多条RUN命令进行合并可以有效降低构建的镜像的层数。



基本原则2： 使用&&连接多条命令。

> 出错之后立即返回Dockerfile中进行错误控制的返回信息，这样不但能快速定位到错误的地方，还能出错时减少镜像构建的时间。在RUN命令中如果需要使用多条语句，建议使用&&进行连接，单条语句结束时使用 \进行连接避免可读性太差。全部使用&&符的好处在于：出错后，后续内容就会不再执行，在镜像构建失败的时候根据出错的第一现场能更加容易地快速确认问题的原因。



### ADD

使用方法：`ADD <src> <dest>`

以Dockerfile所在目录为当前目录，将容器外的文件发送到容器内部。

假定Dockerfile所在目录下有`a.cpp`文件。

```dockerfile
ADD a.cpp .  # 将外部的a.cpp传递给容器内部的 / 目录。关于目录相关的内容参考WORKDIR
```





### WORKDIR

WORKDIR为Dockerfile中的任何RUN、CMD、ENTRYPOINT、COPY和ADD指令设置工作目录。如果Dockerfile中没有指定WORKDIR，则默认是容器内的`/`。注意，如果我们基于某个镜像编写Dockerfile，那么WORKDIR可能不是/。建议用户指定WORKDIR。

WORKDIR指令可以在Dockerfile中多次使用。如果提供了相对路径，则它将与上一个WORKDIR指令的路径相对。

假定当前dockerfile目录下有a.cpp和b.cpp文件，并且Dockerfile的内容为：

```dockerfile
FROM centos:7
WORKDIR /apps
ADD a.cpp .
WORKDIR doc
ADD b.cpp .
```

执行后会在容器的`/apps`下创建`a.cpp`，在`/apps/doc`下创建`b.cpp`。



### ENTRYPOINT

不是很了解~



### FROM

必须写在Dockerfile的开头。每个Dockerfile都必须基于某个镜像构建新的镜像。



### EXPOSE

EXPOSE指令通知Docker容器在运行时对外暴露的端口号。默认暴露TCP端口。如果我们想在外部通过容器暴露的端口向容器发送信息，需要在`docker run`指定中添加`-p`或`-P`相关指令。

```dockerfile
# 暴露1个端口
EXPOSE 8080
# 暴露多个端口
EXPOSE 3000 80 443 22
```





### build

在当前目录下有文件`jdk-8u361-linux-i586.tar.gz`和Dockerfile文件，Dockerfile的内容如下：

```dockerfile
FROM centos:7

WORKDIR /

RUN yum install -y vim && \
yum install -y net-tools && \
yum install -y glibc.i686
ADD jdk-8u361-linux-i586.tar.gz /usr/local/java

# 配置Java的环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_361
ENV JRE_HOME /$JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80

# run 容器后自动执行该命令
CMD /bin/bash

```

执行命令：`docker build -t myrepo/myos:1.0 .`





[参考文章](https://zhuanlan.zhihu.com/p/529732301)

问题：每创建一个新的docker，使用`ifconfig`查看会多一个网卡设备吗？

问题：docker的网桥结构图？

问题：docker默认网桥“接管”所有镜像创建的容器吗？

问题：尝试在“网桥”网络中ping不同的容器（可能是不同镜像生成的容器）。

问题：不同网桥连接的容器可以相互通信吗？



<u>使用 dockerfile 制作镜像时 docker client 会发送 dockerfile 同级目录下的所有文件到docker daemon。解决办法有两种：</u>

- <u>使用 dockerfile 创建镜像时新建目录</u>
- <u>使用 `.dockerignore`，即在 dockerfile 目录下添加 `.dockerignore` 文件，并将需要 ignore 的文件名添加到这个文件中</u>
- <u>删除不需要的文件</u>

<u>我不太懂什么叫做把文件发送给docker daemon。</u>

但是建议使用 dockerfile 创建镜像时新建目录。



## docker 实战

### 案例一：使用centos镜像

1. 首先我们想在远程仓库（Docker Hub）中搜索centos镜像是否存在，需要输入`docker search centos`，之后系统会返回很多相关镜像。可以看到列表中有NAME为centos的镜像。
2. 下载（拉取）Centos镜像：`docker pull centos:7`，指定版本号为7。
3. 运行这镜像，输入`docker run -it --name myos centos:7 /bin/bash`。此时成功运行容器并进入容器。`-i` 表示用户要和容器持久性的interactive，`-t`表示让容器创建一个终端，`/bin/bash` 表示创建容器后立刻运行 /bin/bash 文件。（一般来说，用到-it，就会用到/bin/bash）。`--name server`表示这个容器的名称为myos。
4. 当我们想要退出容器时，按下`ctrl + p + q`或`ctrl + d`即可。（建议第一次按`ctrl + p + q`，之后按`ctrl + d`）
5. 当我们想要再次进入该容器时，先确保它处于运行状态，然后输入`docker exec -it myos /bin/bash`（myos是该容器的名称）。

备注：不建议使用`docker attach`。



### 案例二：使用Tomcat镜像

1. 首先我们想在远程仓库（Docker Hub）中搜索Tomcat镜像是否存在，需要输入`docker search tomcat`，之后系统会返回很多相关镜像。可以看到列表中有NAME为tomcat的镜像。

2. 下载（拉取）Tomcat镜像：`docker pull tomcat`，这种方式通常拉取的是最新版镜像，相当于`docker pull tomcat:latest`。
3. 运行这个镜像，由于需要给该容器开放网络端口，因此还需要建立端口映射，输入`docker run -d -p 50001:8080 --name server tomcat:latest `，注意这里没有`/bin/bash`。此时成功运行容器并获得容器ID。`-d` 表示后台运行这个程序（不用太纠结，**一般来说没有使用-it，就要使用-d**），`-p 50001:8080`表示将宿主机的50001端口映射到容器的8080端口，`--name server`表示这个容器的名称为server。 
4. 注意开放服务器的端口，访问web服务即可。
5. 成功创建好web服务以后，接下来我们要进入这个容器，修改webapps目录中的内容，应该使用`docker exec -it server /bin/bash`指令（server对应容器名）。
6. 修改完毕后，我们使用`ctrl + d`退出容器（推荐方法），也可以使用`ctrl + p + q`退出容器。



### 案例三：容器导出为镜像

1. 当我们在centos容器中安装了vim等工具以后，我们想将该容器导出为一个镜像，方便以后使用。我们需要输入`docker commit myos mycentos:1.0`，myos是镜像名，mycentos是仓库名，1.0是版本号。
2. 输入`docker images`即可查看镜像mycentos。



### 案例四：让容器一直运行

1. 当重启docker后，容器默认不会自动运行，我们需要输入`docker start CONTAINER`才能启动容器。
2. 如果我们想让该容器在docker重启后自动运行，那么使用`docker run`创建容器时需要添加`--restart always`。



### 案例五：将容器导出为tar包

1. 当我们在容器中配置好环境以后，想将其导出为tar包并分享给他人，我们需要输入`docker export -o file.tar server`，此时容器被导出为file.tar并存放在当前目录下，`server`是正在运行的容器名。
2. 别人拿到tar包以后，想要加载tar包为镜像，仅输入`docker import file.tar myserver:1.0`即可。其中myserver是镜像名，1.0是版本号，这两个可以随意指定。



### 案例六：将容器导为镜像

1. 当我们在容器中配置好环境后，想将其上传到docker hub，需要首先制作成镜像。此时需要输入`docker commit -m="添加了vim工具" -a="longcoding" CONTAINER xlongcoding/myos:1.0`。CONTAINER代指容器名，`xlongcoding/myos`叫做仓库名（我习惯将其设置为【作者/软件名】），`1.0`是版本号。



### 案例七：将镜像上传到dockerhub，并下载

1. 我们需要去docker hub官网的上注册账号，并使用`docker login`登录。
2. 我们需要在docker hub的repositories中创建一个仓库。【仓库名】建议命名为【人名/软件名】
3. 我们需要将本地的镜像打一个标签，并且标签名和远程【仓库名】需要一样。 使用`docker push 仓库名:版本号`上传本地镜像。版本号随意指定。



### 案例九：部署Redis应用

[参考文档](https://hub.docker.com/_/redis)

1. 搜索远程仓库中关于Redis的镜像：`docker search redis`，并拉取Redis镜像：`docker pull redis:latest`。

2. 运行容器：

   ```shell
   docker run -id --name my_redis -p 26379:6379 redis --requirepass "你的密码"
   ```

   注意密码需要用双引号括起来。



### 案例十：安装ssh

背景：基于centos7系统，docker的centos7镜像配置ssh工具。

步骤：

1. 创建并运行容器：`docker run -itd --name worker2 --hostname worker2 -p 40001:22 --privileged centos:7 /usr/sbin/init`。

   注意：这里必须要使用`/usr/sbin/init`。

2. 进入容器：`docker exec -it worker2 /bin/bash`

3. 安装依赖：`yum install passwd openssl openssh-server openssh-clients -y`

4. 修改密码：`passwd`

5. 修改配置：

   ```
   vi /etc/ssh/sshd_config
   PubkeyAuthentication yes #启用公钥私钥配对认证方式
   AuthorizedKeysFile .ssh/authorized_keys #公钥文件路径
   PermitRootLogin yes #root能使用ssh登录
   ```

6. 重启ssh服务，并设置开机启动：

   ```
   systemctl restart sshd
   systemctl enable sshd.service
   ```





## docker compose 容器编排

Compose是docker公司推出的一款软件，可以管理一个应用的多个docker容器。我们需要定义一个YAML格式的配置文件`docker-compose.yml`，写出多个容器之间的调用关系。然后，只需要一个命令，就可以关闭/启动这个应用下的多个容器。



### 如何安装

按照[官方文档](https://docs.docker.com/compose/install/other/#on-linux)的步骤即可。由于github访问较慢，可以找一个[国内镜像源](https://get.daocloud.io/#install-compose)



### 如何卸载

必须使用上述安装方式，卸载时输入`sudo rm /usr/local/bin/docker-compose`即可。



### 核心概念

> 一个文件：docker-compose.yml

文件中定义一个工程。



> 两个要素：服务和工程

服务：一个个应用容器实例，例如MySQL容器，订单微服务......

工程：一组关联的应用容器组成一个完整业务单元。



## 其他

```
docker load -i docker的tar包

docker run -p 20000:22 -p 8000:8000 --name django_server -itd django_lesson:1.0

ssh-keygen：git基于ssh
```



```
docker load -i docker的tar包
docker run -itd -p 80:9999 -p 3306:3306 --name csp_server 镜像名:版本


php start.php start -d
php start.php stop
```



docker load -i docker的tar包
docker run -itd -p **80**:9999 -p 3306:3306 --name csp_server 镜像名:版本


php start.php start -d·





启动Nginx：docker run -d -p 50001:80 --name n-server nginx:latest





下载软件：

docker pull xlongcoding/csp_system:1.0

运行软件（注意需要设定端口号）：

docker run -it -p 自定义web服务端口:9999 -p 自定义数据库端口:3306 --name server xlongcoding/csp_system:1.0 /bin/bash

cd /www/wwwroot/cspsystem/

php start.php start -d

/etc/init.d/mysqld start

redis-server /etc/redis.conf



数据库名：cspsystem 

数据库密码：pbKRXJHGAiX7sr6b
