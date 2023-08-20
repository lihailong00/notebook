# vant-weapp使用建议

[toc]



## 写在前面

一定要熟读官方文档！网上关于vant-weapp的教程质量不高。



## 样式覆盖

[官方文档](https://vant-contrib.gitee.io/vant-weapp/#/custom-style)

自定义component中引入vant组件，如果想要改变它的样式，不仅要在自定义组件wxss中设定样式，也要在自定义组件的js文件中添加如下代码：

```js
Component({
  options: {
    styleIsolation: 'shared',
  }
})
```



## 动态切换主题

[官方文档](https://vant-contrib.gitee.io/vant-weapp/#/theme#ding-zhi-dan-ge-zu-jian-de-zhu-ti-yang-shi)



将样式写入js，即可动态切换样式。

案例代码：

> my-test.page

```html
<van-button style="{{ buttonStyle }}" bindtap="changeColor">
  按钮
</van-button>
```



> my-test.js

```js
Page({
  changeColor() {
    this.data.color ^= 1
    if (this.data.color == 0) {
      this.setData({
        buttonStyle: `
        --button-border-radius: 10px;
        --button-default-color: green;
        `
      })
    }
    else {
      this.setData({
        buttonStyle: `
        --button-border-radius: 20px;
        --button-default-color: red;
        `
      })
    }
  },
  data: {
    color: 0,
    buttonStyle: `
      --button-border-radius: 10px;
      --button-default-color: green;
    `,
  }
});
```



## 自定义组件样式

可惜的是vant weapp官方并没有给出每个组件可以修改哪些样式属性。

> my-test.wxml

```html
<view class="container">
  <van-button>
    默认按钮
  </van-button>
</view>
```



> mytest.wxss

```css
.container {
  --button-border-radius: 10px;
  --button-default-color: #f2f3f5;
  --toast-max-width: 100px;
  --toast-background-color: pink;
}
```

