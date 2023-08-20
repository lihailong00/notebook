

# UniApp使用手册



[toc]



## 创建项目

[官方教程](https://uniapp.dcloud.net.cn/quickstart-hx.html)

完全按照官网教程走即可，非常简单。



## 页面结构

[官方教程](https://uniapp.dcloud.net.cn/tutorial/project.html)

不用全部掌握，只需要知道`components`、`pages`、`static`、`manifest.json`、`pages.json`、`uni.scss`即可。

备注：`components`里的文件结构按照[easycom 规范](https://uniapp.dcloud.net.cn/collocation/pages.html#easycom)。



## 配置路由

1. 给`"pages"`数组添加对象`{ "path": "xxxx" }`。
2. 创建`"tabBar"`数组，并在里面填写页面具体信息。

```json
{
  "pages": [ //pages数组中第一项表示应用启动页，参考：https://uniapp.dcloud.io/collocation/pages
    // 第一项路由
    {
      "path": "pages/index/index"
    },
    // 第二项路由
    {
      "path": "pages/home/home"
    }
  ],
  "tabBar": {
    "list": [{
        // pagePath必写，其他选写（建议写上）
        "pagePath": "pages/index/index",
        "text": "首页",
        "iconPath": "static/logo.png",
        "selectedIconPath": "static/logo.png"
      },
      {
        "pagePath": "pages/home/home",
        "text": "我的",
        "iconPath": "static/logo.png",
        "selectedIconPath": "static/logo.png"
      }
    ]
  },
  // 一定要添加，路由才能生效
  "uniIdRouter": {}
}
```



## 全局样式

根目录下的`uni.scss`文件中配置全局样式。

使用`uni.scss`文件的一些注意事项，在创建该文件的时候会自动生成，下面我展示官方编写的注意事项。

```scss
/**
 * 这里是uni-app内置的常用样式变量
 *
 * uni-app 官方扩展插件及插件市场（https://ext.dcloud.net.cn）上很多三方插件均使用了这些样式变量
 * 如果你是插件开发者，建议你使用scss预处理，并在插件代码中直接使用这些变量（无需 import 这个文件），方便用户通过搭积木的方式开发整体风格一致的App
 *
 */

/**
 * 如果你是App开发者（插件使用者），你可以通过修改这些变量来定制自己的插件主题，实现自定义主题功能
 *
 * 如果你的项目同样使用了scss预处理，你也可以直接在你的 scss 代码中使用如下变量，同时无需 import 这个文件
 */
```



## Color UI 组件库的使用

[DCloud网站](https://ext.dcloud.net.cn/plugin?id=239)上下载该项目，运行该项目即可看到Color UI的所有组件。

将下载好的文件中的`colorui`文件复制到自己项目的根目录`/`下。在





## 实战

### 设计一个不跨page的菜单切换功能

```vue
<template>
  <view>
    <view class="flex-box">
      <view :class="['box', {'dark': isActive === 1}]" @tap="tapMenu(1)">
        菜单1
      </view>
      <view :class="['box', {'dark': isActive === 2}]" @tap="tapMenu(2)">
        菜单2
      </view>
      <view :class="['box', {'dark': isActive === 3}]" @tap="tapMenu(3)">
        菜单2
      </view>
    </view>
    <view class="content" v-show="isActive === 1">
      内容1
    </view>
    <view class="content" v-show="isActive === 2">
      内容2
    </view>
    <view class="content" v-show="isActive === 3">
      内容3
    </view>
  </view>
</template>

<script>
  import {
    ref
  } from 'vue'
  export default {
    setup() {
      let isActive = ref(1);
      const tapMenu = (signal) => {
        isActive.value = signal;
      };
      return {
        isActive,
        tapMenu
      }
    }
  }
</script>

<style scoped lang="scss">
  .flex-box {
    display: flex;
    justify-content: center;
    width: 100%;
    margin-bottom: 20upx;
  }

  .box {
    width: 180upx;
    height: 100upx;
    background-color: #89BCCC;
    margin-left: 20upx;
    text-align: center;
    line-height: 100upx;
    color: white;
  }

  .dark {
    background-color: #5b6478;
  }

  .content {
    width: 600upx;
    height: 300upx;
    background-color: #b3b3b3;
    margin: 0 auto;
  }
</style>
```

