# vue router使用手册

[toc]

## 什么是vue router

Vue创建的前端项目是单页面应用。也就是说，用户请求服务器，服务器一次性将多个组件和页面合并在一起，返回给用户。当用户更新视图时，不会再次请求服务器，而是通过vue-router实现页面的跳转。



## 使用方法


### 新建vue项目

首先我们需要创建一个vue项目，这里我使用vite构建工具。参考[官方文档](https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project)。

1. 使用npm安装vite：`npm create vite@latest`。注意使用npm前需要[修改镜像源](https://blog.csdn.net/m0_64409387/article/details/124141529)。
2. 接下来按照命令行的提示一步步执行即可。创建完毕后删掉`App.vue`中多余的文件和`components`中的vue文件。
3. 在`components`中创建3个文件：`a.vue`、`b.vue`、`c.vue`、`d.vye`。并在`<template>`标签中填写各自的内容。

### 使用vue-router

[官方文档](https://router.vuejs.org/zh/installation.html#npm)

1. 使用npm安装vue-router：`npm install vue-router@4`。

2. 在`/src`目录下创建文件`/router/index.js`，并填写以下代码：

   ```js
   import { createRouter, createWebHistory } from "vue-router"
   
   const routes = [
       {
           path: '/a',
           component: () => import('../components/a.vue')
       },
       {
           name: 'b',
           path: '/b',
           component: () => import('../components/b.vue')
       },
       {
           path: '/c',
           component: () => import('../components/c.vue')
       },
       {
           name: 'd',
           path: '/d',
           component: () => import('../components/d.vue')
       }
   ]
   
   const router = createRouter({
       // 路由模式有两种：createWebHistory和createWebHashHistory
       // 前者的路径中没有#，后者有。#后路径改变不会引起页面刷新。 
       history: createWebHistory(),
       routes
   })
   
   export default router
   ```

3. 在`/src/main.js`中引入`router`对象，并使用它：

   ```js
   // /src/main.js
   import { createApp } from 'vue'
   import App from './App.vue'
   // 核心代码
   import router from "./router/index.js";
   
   // 注意这里也要加
   createApp(App).use(router).mount('#app')
   ```

4. 在`/src/App.vue`中添加如下代码，即可通过网址栏实现路由：

   ```vue
   <script>
   </script>
   
   <template>
     <router-view></router-view>
   </template>
   
   <style scoped>
   </style>
   ```

5. 通过`<router-link>`实现页面跳转，在`/src/App.vue`中继续添加代码：

   ```vue
   <script setup>
   import {useRouter} from "vue-router";
   
   const router = useRouter()
   
   const toPage1 = (url) => {
     // 通过url路由
     router.push({
       path: url
     })
   
     // 如果改成replace，就没有历史记录
     // router.replace({
     //   path: url
     // })
   }
   
   const toPage2 = (name) => {
     // 通过名称路由
     router.push({
       name: name
     })
   }
   </script>
   
   <template>
     <!--  通过路径跳转-->
     <router-link to="/a">a</router-link>
   <!--  如果加上replace属性，则不会有历史记录-->
   <!--  <router-link to="/a" replace>a</router-link>-->
     <br />
     <!--  通过路由跳转-->
     <router-link :to="{name: 'b'}">b</router-link>
     <br />
     <button @click="toPage1('/c')">c</button>
     <br />
     <button @click="toPage2('d')">d</button>
     <br />
     <router-view></router-view>
   </template>
   
   <style scoped>
   </style>
   ```
   
   其中页面跳转有3种方式：通过路径跳转、通过name跳转和通过函数跳转。如果想跳转后没有历史记录，增加replace属性即可。



### 路由传参

1. 路由传参有两种方式：一种是使用`query`，参数在url中；另一种是使用`params`，参数隐藏。

2. 这里不建议使用params传参，其中一个原因是页面刷新会丢失数据，个人习惯用pinia存放数据。[参考文档](https://github.com/vuejs/router/blob/main/packages/router/CHANGELOG.md#414-2022-08-22)。

3. 具体操作：

   1. 创建数据文件`/src/data.js`，并填写代码：

      ```js
      const user = {
          name: 'lhl',
          age: 8,
          gender: 'male'
      }
      
      export { user }
      ```

      

   2. 在`/src/App.vue`中增加代码：

      ```vue
      <script setup>
      import {useRouter} from "vue-router";
      // 引入数据
      import {user} from "./static/data.js";
      
      const router = useRouter()
      
      const toPage1 = (url) => {
        router.push({
          path: url
        })
      }
      
      const toPage2 = (name) => {
        router.push({
          name: name
        })
      }
      
      const toDetail = (user) => {
        router.push({
          path: '/detail',
          query: user
        })
      }
      </script>
      
      <template>
        <router-link to="/a">a</router-link>
        <br />
        <router-link :to="{name: 'b'}">b</router-link>
        <br />
        <button @click="toPage1('/c')">c</button>
        <br />
        <button @click="toPage2('d')">d</button>
        <br />
      
      <!--  路由传参-->
        <button @click="toDetail(user)">去详情页面</button>
        <br />
        <router-link :to="{name: 'detail', query: {name: user.name, age: user.age, gender: user.gender}}">去详情页面</router-link>
        <br />
        <router-view></router-view>
      </template>
      
      <style scoped>
      </style>
      ```

      

   3. 创建组件`/src/components/Detail.vue`，并填写代码：

      ```vue
      <template>
        <div>详情界面</div>
        <div>姓名：{{route.query.name}}</div>
      <!--  也可以通过this.$router取出参数中的数据-->
        <div>年龄：{{this.$route.query.age}}</div>
        <div>性别：{{route. query.gender}}</div>
      </template>
      
      <script setup>
      import {useRoute} from "vue-router";
      
      const route = useRoute()
      console.log(route)
      </script>
      
      <style scoped>
      
      </style>
      ```

   4. 编写路由文件：`/src/router/index.js`

      ```js
      {
          name: 'detail',
          path: '/detail',
          component: () => import('../components/Detail.vue')
      }
      ```



### 嵌套路由



