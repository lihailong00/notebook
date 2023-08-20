

# eslint

[toc]

## 参考文章

[文章](https://juejin.cn/post/6990929456382607374)



## 安装eslint

1. 本地初始npm

   ```bash
   npm init -y
   ```

   

2. 本地安装并启动eslint

   ```bash
   npx eslint --init
   ```

   关于`npx`，参考另一个文档。

   

4. 自定义选择配置eslint。

4. 创建完eslint后，它的纠错规则是什么呢？软件自带一些配置。同时也可能需要人为配置，进入`.eslintrc.js`文件。（eslint也可以是yaml或json格式）

   ```js
   module.exports = {
       "env": {
           "browser": true,
           "es2021": true
       },
       "extends": "eslint:recommended",
       "overrides": [
       ],
       "parserOptions": {
           "ecmaVersion": "latest",
           "sourceType": "module"
       },
       "rules": {
       }
   }
   ```

   `rules`以外的配置都是上一步设定的（不用在乎它们）。

   `rules`中的内容才是重点！如何配置rules中的内容呢？谷歌或根据自己的项目。

   

   属性所代表的值：**0表示忽略，1表示警告，2表示报错**

   ```js
   "rules": {
       "quotes": 2,
       "semi": 2
   }
   ```

   

6. 假定我有一个`index.js`，如何使用eslint检查`index.js`的错误呢？并且能自动纠错吗？都可以！

   ```js
   let a = 12
   console.log('hello');
   ```

   输入命令：`eslint index.js`

   结果如下：

   ```
   1:11  warning  Missing semicolon	semi
   2:13  error    Strings must use doublequote  quotes
   ```

   使用纠错：`eslint index.js --fix`

   

   备注：上述案例仅用于演示eslint的使用方法。（一般脑子有病才用上面的配置）



## eslint插件

上述使用eslint的案例有一个缺点：不能一边写代码，一边自动提示错误并修改。这是需要用到eslint插件。

安装好eslint插件，也就配置好eslint的基本规范。但是会优先执行自己配置`.eslintrc.js`文件。



## prettier

```
npm i prettier -D
```

自己创建`.prettierrc.js`

格式化`index.js`文件

```
npx prettier --write index.js
```



prettier不仅可以格式化js文件，还可以格式化html和css文件。而eslint只能格式化js文件。



**注意：prettier和eslint的配置不要冲突！**



同理：如果我们想让代码自动保存后使用prettier格式化，我们需要使用prettier插件。



## 实际使用

一般来说，使用脚手架创建好项目后，会自带`.eslintrc`和`.prettierrc`文件。我们一般来说也不需要修改这两个文件。我们可能只需要安装eslint插件和prettier插件即可。