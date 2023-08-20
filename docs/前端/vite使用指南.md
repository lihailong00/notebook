# vite使用指南

[toc]



## vite是什么

vite是一个现代化的前端构建工具，可以帮我们做很多事。

> 1. 支持Vue、React等框架的开发。
> 2. 提供开箱即用的服务器。
> 3. 支持语法降级。（babel）
> 4. 支持lass、sass......
> 5. 支持直接从node_modules中引入代码
> 6. 打包

**我们只需要写代码即可，不用关心编译过程**。





## 生产环境和开发环境的配置

在我的项目中，区分生产环境和开发环境的主要作用是**配置不同的后端地址**，在`package.json`所在的同级目录下，分别创建`.env.development`和`.env.production`文件，并添加如下内容。

> .env.development

```
ENV = 'development'

VITE_BACKEND_URL='http://localhost:8080'
```



> .env.production

```
ENV = 'production'

VITE_BACKEND_URL='http://xxx.xxx.xxx.xxx:xxxx'
```



在使用变量`VITE_BACKEND_URL`的时候，按照该方式：

```vue
import.meta.env.VITE_BACKEND_URL
```

即可使用该变量。