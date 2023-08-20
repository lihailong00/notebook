```
npm init -y
```



```
npm add webpack webpack-cli --dev
```



创建`/src/data.js`，写入

```js
export function getBlogPosts() {
    return ["博客1", "博客2", "博客3"];
}
```

创建`style.css`，写入

```css
* {
    margin: 0;
    padding: 0;
    font-family: sans-serif;
}
body {
    display: grid;
    place-items: center;
    height: 100vh;
}
ul {
    list-style: none;
}
li {
    padding: 12px;
}
img {
    max-width: 500px;
}
```



创建`/src/index.js`，写入

```javascript
import { getBlogPosts } from './data';
import "./style.css";  // 不适用loader可能会出错

const blogs = getBlogPosts();
const ul = document.createElement("ul");
blogs.forEach((blog) => {
    const li = document.createElement("li");
    li.innerText = blog;
    ul.appendChild(li);
});
document.body.appendChild(ul);
```





此时，打包css文件还需要两个loader

```
npm add --dev style-loader css-loader
```

注意：几乎所有和webpack相关的依赖都需要安装在开发者环境中，因为打包后不需要它们。





配置webpack：

在`/`目录下创建`webpack.config.js`文件，输入以下内容：

```js
const path = require("path");  // path 是 webpack.config.js文件的路径

// 配置webpack
module.exports = {
    // 开发模式
    mode: "development",
    // 入口文件
    entry: "./src/index.js",
    // 打包后的配置
    output: {
        // 打包后的文件名字
        filename: "dist.js",
        // 打包后文件存放的地址
        path: path.resolve(__dirname, "dist"),
    },
    	
    module: {
        // rules中的每一个值都对应loader中的配置，其中包含匹配拓展名和使用哪些loader
        rules: [
            {
                // test用于匹配文件名
                test: /\.css$/i,
                use: ["style-loader", "css-loader"]
            }
        ]
    }
};
```





```
npx webpack
npx 可以自动运行 node_modules 里自带的库的命令。
```

多了个`/dist/dist.js`



创建`/index.html`，写入

```html
<!--引入打包后的文件-->
<script src="./dist/dist.js"></script>
```



以上就是一个完整的打包过程。将`/src/index.js`以及里面引入的相关库全部打包成`dist.js`文件。



```
npm install html-webpack-plugin --dev
```

配置webpack.config.js

```
npx webpack
```



```
npm install --dev babel-loader @babel/core @babel/preset-env
```



配置webpack.config.js

```
npx webpack
```



压缩打包后的文件

```
npm install --dev terser-webpack-plugin
```



配置webpack.config.js
