# CentOS7软件包管理

[toc]

## RPM

### RPM是什么

RPM是一款管理软件包的程序。管理RPM包。RPM包是格式为.rpm的软件包。

### RPM包格式

`nginx-1.23.0-1.1.x86_64.rpm`

nginx：软件名称。

1.23：主版本号.副版本号

x86_64：64位操作系统。

其余不管。



特殊名称：

el7：运行在centos7系统上。

noarch：任何平台均可运行。

注：一个系统可能有多个平台。



### RPM包的获取方式

1. 光盘（我不常用）。
2. [RPM官方网站](rpmfind.net)
3. 各大厂商的官网。



### 安装RPM格式的包

```
rpm -ivh 全名
-i 安装
-v 显示安装信息
-h 显示安装进度
```

**rpm**使用时，什么情况下使用软件包全名，什么时候使用软件包名？

全名：在安装和更新升级时候使用

包名：对已经安装过的软件包进行操作时，比如查找**已经安装**的某个包，卸载包等 ，使用包名。它默认是去目录`/var/lib/rpm`下面进行搜索。 当一个 rpm 包安装到系统上之后，安装信息通常会保存在本地的 `/var/lib/rpm/`目录下。 （`/var/lib/rpm/`这个目录并不放置rpm包，而是放置rpm包的软件）



### RPM的查询

```
rpm -q 常与以下参数组合使用
-a (all)  -->  查询范围是所有的安装包
-f (file) -->  查询文件所属软件包
-i (info)  -->  查询rpm包信息，后面跟rpm包名
-l (list)  -->  查询文件安装位置
-p  -->  查询未安装软件包的相关信息
-R  -->  查询软件包的依赖
```

案例：

```
rpm -q vim  -->  查询是否安装vim软件
rpm -qa  -->  查询所有已安装的软件
rpm -qa | grep vim  -->  查询已安装软件中带有"vim"的软件
# which find  -->  查找find指令所在的位置
rpm -qf /usr/bin/find  -->查看find文件所属的软件包
rpm -qi mysql80-community-release-el7-6.noarch.rpm  -->  显示已安装的rpm包的信息
rpm -qpi mysql80-community-release-el7-6.noarch.rpm   -->  显示未安装的rpm包的信息
rpm -qpl mysql80-community-release-el7-6.noarch.rpm  -->  查看安装rpm包后，生成哪些文件
rpm -V findutils  -->  查看findutils包安装的软件是否被篡改
rpm -Vf /usr/bin/find  -->  查看find文件是否被篡改
```

### RPM卸载和更新

**建议使用yum命令更新或卸载**

rpm -e 包名

```
rpm -e wget  -->  卸载wget包
rpm -e --nodeps wget  -->  忽略wget的依赖，强制卸载
```



## YUM

### YUM是什么

YUM类似RPM，都是**包管理器**，只是YUM可以**自动处理依赖关系，并且可以一次性安装所有具有依赖关系的软件包**。



### 配置YUM源

**先说技巧：输入以下指令即可：**

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```



再说说大致原理。

YUM源就是通过yum安装软件的软件安装包的来源。我经常使用网络搭建yum源。以下是网络搭建yum源的方法。

yum源的配置文件都位于`/etc/yum.repos.d/`里，是后缀为`.repo`的文件。

通常名为`CentOS-Base.repo`的文件生效，打开该文件：

```
# 括号里的是容器名称
[base]  
# 容器的说明，随便写
name=CentOS-$releasever  
# 该容器生效。如果值为0，则不生效。
enabled=1  
failovermethod=priority
# yum源服务器地址
# $releasever表示当前系统版本，可通过命令'cat /etc/centos-release'查看
# 如果yum源不可用，尝试将$releasever替换为centos7的最新版本
# $basearch 通常等于：x86_64
baseurl=http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/  
# RPM的数字证书生效
gpgcheck=1  
# 数字证书的公钥文件保存位置
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-7  

[updates]
name=CentOS-$releasever
enabled=1
failovermethod=priority
baseurl=http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever
enabled=1
failovermethod=priority
baseurl=http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-7
```

备注：个人认为，[]中的名字似乎可以任意命名，但是最好不要修改。

### YUM的常见操作

```
直接安装nginx软件
yum install -y nginx

卸载已安装的软件
yum -y remove  包名

升级软件，改变软件设置和系统设置,系统版本内核都升级
yum -y update  

升级软件包，不改变软件设置和系统设置，系统版本升级，内核不改变
yum -y upgrage

查看当前系统有哪些【软件组】，每个软件组有很多软件
yum grouplist

安装名为'Development Tools'的软件组
yum groupinstall 'Development Tools'  -y

查询rpm包所属的软件组
rpm -qpi mysql80-community-release-el7-6.noarch.rpm

nginx软件的作用
yum info nginx

查询find命令是哪个软件包安装的
yum provides /usr/bin/find

查询含"nginx"关键字的软件包
yum search nginx
```



## 源码安装（未完待续）

