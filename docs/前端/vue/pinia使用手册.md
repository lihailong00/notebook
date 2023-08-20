# pinia使用手册

[toc]

## 参考文章

[大菠萝！这一次彻底搞懂Pinia！（保姆级教程）](https://juejin.cn/post/7112691686085492767)



## 使用pinia

1. 使用vite创建项目：`npm create vite@latest`。

2. 安装pinia：`npm install pinia`。

3. `/src/main.ts`中挂载Vue应用：

   ```typescript
   // /src/main.ts
   // main.ts
   import { createApp } from "vue";
   import App from "./App.vue";
   import { createPinia } from "pinia";
   const pinia = createPinia();
   
   const app = createApp(App);
   app.use(pinia);
   app.mount("#app");
   ```

4. 创建store。

   ```typescript
   // /src/store/user.ts
   
   import {defineStore} from "pinia";
   
   // 命名规则：use + 名字 + Store
   // 第一个参数是id，第二个参数是配置参数，按照模仿即可。
   export const useUserStore = defineStore('main', {
       state: () => {
           return {
               name: 'lhl',
               age: 8,
               score: 100
           }
       }
   })
   ```

5. 接下来我们直接使用store即可，下面是一个使用案例，文件地址：`/src/App.vue`。

   ```vue
   <script setup lang="ts">
   import {useUserStore} from "./store/user"
   import {ref} from "vue"
   const store = useUserStore()
   const name = ref<string>(store.name)
   const age = ref<number>(store.age)
   const score = ref<number>(store.score)
   // 也可以通过解构的方式获取store中的值
   // const { name, age, score } = store
   
   // 如果想要响应式修改值，则按照下述方法获取store中的值
   // const {name, age, score} = storeToRefs(store)
   
   const addScore = () => {
     store.score++
   }
   
   const reset = () => {
     store.$reset()
   }
   </script>
   
   <template>
     <div>姓名：{{name}}</div>
     <div>年龄：{{age}}</div>
     <div>分数：{{score}}</div>
   <button @click="addScore">分数+1</button>
   <button @click="reset">重置数据</button>
   </template>
   
   <style scoped>
   </style>
   ```

