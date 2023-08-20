# Nginx使用手册

[toc]

## 写在前面



## 安装并操作nginx

1. 直接使用docker容器关于nginx的镜像。

​	建议在docker容器中安装nginx，并让docker容器内部的一个端口和外部服务器的一个端口对应。

参考如下命令：

`docker run -it -p 8888:80 --name ng nginx /bin/bash`



2. 基于Linux Ubuntu系统。
   1. 使用root权限的用户。
   2. 更新软件包：`apt-get update`。
   3. 安装Nginx：`apt-get install -y nginx`。
   4. 启动nginx服务：`service nginx start`或`systemctl start nginx`。（初始化的docker容器中没有`systemctl`命令）
   5. 查看nginx是否工作：`service nginx status`或`systemctl status nginx`。
   6. 停止运行nginx：`service nginx stop`或`systemctl stop nginx`。
   7. 删除nginx：
      1. 停止nginx服务：`service stop nginx`。
      2. 删除nginx包：`apt-get remove -y nginx`。
      3. 删除nginx的配置文件：`apt-get purge nginx`。
      4. 删除Nginx依赖项和无用的软件包：`apt-get autoremove -y`
      5. 如果想删掉nginx的日志，执行`rm -rf /var/log/nginx`。




3. 基于CentOS系统。
   1. 使用root权限的用户登录。
   2. 安装 EPEL 软件源，这是一个 CentOS 第三方软件源，可以提供更多的软件包。在终端中运行以下命令：`yum install epel-release`。（由于EPEL比较慢，建议改成国内的镜像源）
   3. 安装nginx：`yum install nginx`。
   4. 安装完成后，启动 Nginx 服务并设置开机自启。在终端中运行以下命令：`systemctl start nginx`和`systemctl enable nginx`。
   5. 验证 Nginx 是否已成功安装并正在运行。在浏览器中输入服务器的 IP 地址或域名，如果可以看到 Nginx 的欢迎页面，说明 Nginx 已经成功安装并运行。




4. Windows中安装。

   1. 下载NGINX：

      1. [官网](https://nginx.org/en/download.html)下载NGINX的Windows版本。
      2. 下载并解压后，**不建议**配置环境变量，<u>建议每次到NGINX的解压目录使用cmd</u>。

   2. 使用NGINX：

      1. 启动：`start nginx`
      2. 处理完当前的请求后关闭：`nginx -s quit`。
      3. 立即关闭：`nginx -s stop`。
      4. 修改完配置文件后重新加载：`nginx -s reload`
      5. 查看NGINX版本：`nginx -v`




## nginx请求流程





## nginx的配置

1. **使用docker拉取nginx镜像**，并运行容器后，输入`whereis nginx`查看关于nginx的地址。
2. nginx的配置文件通常在`/etc/nginx`下。



### /etc/nginx/nginx.conf

下面是nginx的默认配置，之后我会解释每个字段的作用。

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/ *.conf; # 注意/* 之间没有空格

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
```



1. `worker_processes`：最开始开启的**工作进程**数量。建议一个cpu内核对应一个进程。

2. `events { worker_connections 768 }`：每一个进程对应多少个连接。

3. http模块

   ```nginx
   http {
       # 引入外部配置文件
       # 当一个客户端向 Nginx 服务器请求一个资源时，Nginx 会检查文件扩展名，并使用 mime.types 文件中的映射关系来确定响应类型。
       # 如果找不到匹配的映射，则默认情况下 Nginx 会返回 application/octet-stream，这通常意味着客户端无法处理响应的数据。
       include /etc/nginx/mime.types;
       
       # 如果文件类型不包含在 include 中，则以该方式将数据返回给浏览器
       default_type application/octet-stream;
       
       # 启用 sendfile 可以显著提高 Nginx 的性能，但在某些情况下可能会导致问题。
       sendfile on;
       
       # 当启用 keep-alive 连接时，客户端可以在同一连接上发出多个请求，而无需每次都建立新的 TCP 连接。
       # 需要注意的是，如果 keepalive_timeout 设置得太小，可能会导致客户端在发送下一个请求之前必须重新建立连接，这可能会对性能产生负面影响。另一方面，如果 keepalive_timeout 设置得太大，可能会导致服务器资源被长时间占用，从而限制并发连接数。
       # 单位是 秒
       keepalive_timeout 65;
       
       # 引入外部配置文件
       # conf.d 目录下存放nginx的全局配置文件，例如服务器性能、安全性、SSL 证书等相关的设置。
       include /etc/nginx/conf.d/*.conf;
   
       # sites-enabled 目录下存放 Nginx 虚拟主机的配置文件。这些虚拟主机配置文件通常用于指定不同的网站或应用程序，包括其域名、端口、SSL 证书、反向代理等相关的设置。
       include /etc/nginx/sites-enabled/*;
   }
   ```

4. 虚拟主机的配置

   在 Nginx 中，虚拟主机是一种用于管理多个网站或应用程序的方式。通过虚拟主机，管理员可以在一台服务器上运行多个网站或应用程序，每个网站或应用程序都有自己的域名、SSL 证书、反向代理等配置。

   虚拟主机的配置从 `server` 块开始。每个 `server` 块表示一个虚拟主机的配置。在 `server` 块中，管理员可以指定虚拟主机的监听端口、域名、SSL 证书等相关的设置。例如：

   ```nginx
   # 对应一个虚拟主机
   server {
       # 不同虚拟主机的监听 端口+主机名 不能完全一样
       listen 80;
   	# 配置 域名或主机名 (可设置多个)
       server_name localhost myhost;
       
       # 对于http://lihailong.vip/doc/index.html来说
       # url是http://lihailong.vip/doc/index.html
       # uri是/doc/index.html
       # location 用于匹配uri
       # 当收到uri为 /data/index.html 的请求后，会到/wwwroot/html目录下查找
       location /data/ {
           # root指定了对外暴露资源的根目录
           root /wwwroot/html;
           # index指定了`/data/`路径映射的内部文件
           index index.html index.htm;
       }
   }
   ```




## 虚拟主机的原理和案例

在 Nginx 中，虚拟主机的实现是基于“server”块和“server_name”指令的。在配置文件中，每个虚拟主机都有一个独立的“server”块，其中定义了该虚拟主机的相关配置信息。每个“server”块都有一个“server_name”指令，用于指定该虚拟主机要响应的域名或IP地址。

当 Nginx 接收到客户端的请求时，会根据请求的主机头（host header）字段匹配相应的虚拟主机。匹配规则是按照配置文件中“server”块出现的顺序进行的。具体过程如下：

1. 当 Nginx 接收到客户端的请求时，会根据请求的主机头字段确定请求的目标虚拟主机。
2. Nginx 会按照配置文件中“server”块出现的顺序，逐一检查每个“server”块的“server_name”指令，直到找到与请求的主机头匹配的“server_name”指令。
3. 一旦找到匹配的“server”块，Nginx 就会使用该“server”块的配置信息来处理客户端请求。

例如，假设我们在 Nginx 的配置文件中定义了两个虚拟主机，分别对应两个域名：[www.example.com](http://www.example.com/) 和 blog.example.com。配置文件如下：

```nginx
http {
  server {
    listen 80;
    server_name www.example.com;
    root /var/www/html/www.example.com;
  }

  server {
    listen 80;
    server_name blog.example.com;
    root /var/www/html/blog.example.com;
  }
}
```

当有用户访问 [www.example.com](http://www.example.com/) 时，Nginx 会根据请求的主机头字段匹配到第一个“server”块，并使用该块的配置信息来处理请求。而当有用户访问 blog.example.com 时，Nginx 会匹配到第二个“server”块，并使用该块的配置信息来处理请求。

这样，Nginx 就可以通过虚拟主机来实现在同一台服务器上托管多个域名或站点的功能。



## 配置 正/反 向代理







