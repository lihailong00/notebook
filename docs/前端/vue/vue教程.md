### 创建vue项目

```
vue create 项目名
```

```
npm run serve
```



### 安装 vant UI (vue3)

```
npm install vant@next -S
```

```
npm i unplugin-vue-components -D
```

安装babel插件

```
npm i babel-plugin-import
```



修改`babel.config.js`文件

```js
module.exports = {
  presets: [
    '@vue/cli-plugin-babel/preset'
  ],
  // 配置vant按需引入
  plugins: [
    ['import', {
      libraryName: 'vant',
      libraryDirectory: 'es',
      style: true
    }, 'vant']
  ],
}
```

