# weui组件库的使用

[toc]

## 快速上手

参考文档：[微信官方文档](https://developers.weixin.qq.com/miniprogram/dev/component/)

随便找一个案例，翻到**示例代码**处，点击**在开发者工具中预览效果**。然后去微信开发者工具中拷贝`weui.wxss`。然后放到自己项目文件的根目录。在`app.wxss`中引入`weui.wxss`（如下），即可使用`weui`的组件。

> app.wxss

```css
@import "/weui.wxss"
```

具体的组件怎么用？去[微信官方文档](https://developers.weixin.qq.com/miniprogram/dev/component/)查看。



备注：`weui`官方文档写的就想一坨***，只能按照官方案例给的代码不断摸索。



## swiper 轮播图

实例代码：

```html
<swiper
  indicator-dots="{{true}}"
  autoplay="{{true}}"
  circular="{{circular}}"
  interval="{{3000}}" 
  duration="{{1000}}" 
  previous-margin="{{0}}rpx" 
  next-margin="{{0}}rpx"
>
  <swiper-item>第1页</swiper-item>
  <swiper-item>第2页</swiper-item>
  <swiper-item>第3页</swiper-item>
</swiper>
```



相关API：[官方文档](https://developers.weixin.qq.com/miniprogram/dev/component/swiper.html)

常用API：`interval`,`duration`,`indicator-dots`,`autoplay`,`circular`,`previous-margin`,`next-margin`。

