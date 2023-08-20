# Vue+SpringBoot+Nginx部署教程

[toc]

## 思路

假定springboot服务的地址是`1.1.1.1:8080`，Nginx的服务地址是`1.1.1.1:80`。

> 前端资源

Vue项目打包后可以得到一个单文件`index.html`和一些静态资源。我们将这些前端资源放在Nginx的指定目录，通过`1.1.1.1:80`即可获取前端资源。

注意，vue是单页面应用，直接通过uri访问资源可能会出现404问题。需要nginx配置。



> 后端资源

我们有两种策略：

1. 客户端直接向`1.1.1.1:8080`发送请求：客户端获取到`1.1.1.1:80`站点的资源，使用JS的`ajax`或`axios`向`1.1.1.1:8080`发送资源时，会产生跨域问题（不懂就百度）。客户端浏览器会自动在请求中添加`Origin`等字段，用于表明该请求是跨域请求，springboot服务收到该跨域请求后，默认会拒绝该请求。解决方式是修改Springboot配置，让Springboot服务接收跨域请求。

   

2. 客户端向`1.1.1.1:80/api`发送请求，然后Nginx将`/api`收到的请求转发给`1.1.1.1:8080`。



## 实战

> 前端配置

1. 开发时，前端请求后端让代理去做。下面给一个vite的配置案例，在`vite.config.js`中：

```js
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, '')
      }
    }
  }
})
```

但凡涉及到请求后端的接口，都要携带上`/api`前缀，例如：

```js
let courseAddUrl = ref('/api' + "/course/add")
```

需要注意的是：上述配置仅在**开发模式**中有效，其原理是我们配置vite，相当于设置它的配套web服务器**koa**，让koa帮我们转发请求。

2. 然而在生产模式中，vue项目通常打包成一个文件，放在Nginx服务器上，因此我们还需要配置Nginx服务器，让它帮我们实现请求转发。

Nginx配置如下（直接使用就行）：

```nginx
server {
    listen       80;
    server_name  localhost;

    root   html/dist;
    index index.html;

    # 请求代理 两条斜线不能少！
    location /api/ {
        proxy_pass http://localhost:8080/;
    }

    # 设置首页
    location / {
        try_files $uri $uri/ @router;
        index  index.html index.htm;
    }

    # 解决Vue项目404错误
    location @router {
        rewrite ^.*$ /index.html last;
    }
}
```

注意，访问Vue项目的子项目会出现404错误，需要用到上述配置：

```nginx
# 解决Vue项目404错误
location @router {
    rewrite ^.*$ /index.html last;
}
```



## session和cookie（写在cookie篇章可能更好）

同源请求会默认携带cookie，非同源请求需要配置withCredentials为true才能携带cookie。

有些cookie是httponly的，不同通过js操作，提高了安全性。

