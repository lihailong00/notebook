# Linux服务器部署Django



## 前言

根据我多次尝试，不建议在宿主机上直接部署Django，而是拉取一个Python的官方docker镜像，并在Python容器中部署Django，理由如下：

宿主机安装Python不方便，Python 3.10以后需要openssl库的版本大于1.1.1，然而centos的yum源中的openssl版本默认为1.0.7（Ubuntu系统不清楚），而通过源码安装openssl比较麻烦，综合考虑下来，决定在docker容器中运行Django。



## 步骤

### 第一次安装

1. 安装docker最新版。

2. 配置docker镜像源（必应，简单）。

3. 拉取python3.9版本的镜像：``docker pull python:3.9``。不要用高于3.9的版本，应该会有兼容性问题，比如dddocr不兼容高版本python。

4. docker容器对外暴露一个端口，运行Python docker容器。

5. 进入后修改debian镜像源，debian版本应该是12，如果不是，则参考网上的镜像源配置方式。如果是，则按照下面的方式配置debian镜像源。

   1. 进入目录：/etc/apt/sources.list.d

   2. 备份文件debian.sources为debian.sources.bak

   3. 查看debian.sources文件

   4. ```Bash
      Types: deb
      # http://snapshot.debian.org/archive/debian/20231218T000000Z
      URIs: http://deb.debian.org/debian
      Suites: bookworm bookworm-updates
      Components: main
      Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
      
      Types: deb
      # http://snapshot.debian.org/archive/debian-security/20231218T000000Z
      URIs: http://deb.debian.org/debian-security
      Suites: bookworm-security
      Components: main
      Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
      ```

   5. 将````deb.debian.org````替换为````mirrors.tuna.tsinghua.edu.cn````，这里给出一个简单的方法：

   6. ```Bash
      sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apt/sources.list.d/debian.sources
      ```

   7. 

6. 配置pip的镜像源：``pip config set global.index-url ``https://mirrors.aliyun.com/pypi/simple````，配置成功后可以查看一下：``pip config list``

7. 下载vim：``apt-get update`` ``apt-get install vim``。

8. git clone代码。python镜像应该默认带有git工具。但是有时候会遇到github身份验证的情况，如果出现，建议容器生成ssh key并上传到自己的GitHub账户，今后用ssh的方式clone 代码，当然也可以用https的方式clone代码。一般来说GitHub很慢，建议将代码上传到gitee上面。

9. 下载好代码后，需要先安装mysql-client的一些前置工具：``apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config`` ，然后安装mysql-client：``pip install mysqlclient``参考：https://github.com/PyMySQL/mysqlclient

10. 安装requirements.txt中的依赖：``pip install -r requirements.txt``。

11. 修改settings.py中的数据库配置，目前来看，host无法识别``host.docker.internal``，建议填写当前网络的网关到ip（例如172.17.0.1），之后再看看怎么回事。

12. 运行Django：``python manager runserver 0.0.0.0:1234``. 端口可以自己随便指定。

13. prod环境中，不要直接修改代码，尤其是直接修改model代码！而是在dev环境修改后，上传到GitHub，再从生产环境clone下来，并且将dev环境中数据库中的迁移表（django_migrations）一起复制到prod环境中。



### 持续更新

如果之前已经部署成功了，当更新代码时，需要做以下事情。

1. 将代码push到远程仓库，并在生产环境下pull代码。
2. 查看Django进程监控的端口号对应的pid，然后``kill -15``这个进程。
3. 进入项目根目录，``python manage runserver 0.0.0.0:5555``，也可以自定义其他端口号。