# 服务器运行django项目

[toc]

## 本项目做了什么

本项目基于Django，使用playwright自动抓取教务处课表，并返回给前端。



## 前置条件

我使用docker镜像创建了Ubuntu 20.04容器。每个容器中仅运行一个python项目，因此不用venv。

Ubuntu 20.04 默认安装python3.8，如果要安装别的版本，不要参考该文章。



## 安装必备软件

```
apt-get update
apt-get install -y vim
```

更换国内镜像源，备份`/etc/apt/source.list`，并用以下内容替换：

```
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security multiverse
```

```
apt-get update
```



## 安装python项目相关依赖包

```
apt-get install -y python3-pip
pip -V
# 配置清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```



requirements.txt

```txt
ddddocr~=1.4.7
Pillow~=9.4.0
lxml~=4.9.2
playwright~=1.30.0
requests~=2.28.2
Django~=4.1.5
```

pip install -r requirements.txt

playwright install 