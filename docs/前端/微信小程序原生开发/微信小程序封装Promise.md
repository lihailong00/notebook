# 微信小程序初始化操作

[toc]



## 为什么不用原生的wx.request

原生请求会出现回调地狱问题，并且很多时候实现不够优雅。



## 优雅切换环境

项目周期分为开发阶段（develop）和生产阶段（production），不同阶段的配置是不一样的（例如开发环境和生成环境的后端URL不一样）。我们有必要封装“一键切换环境”的操作。

> config.js：用于设置当前开发环境

```js
let env = "develop"  // 可以指定为develop或production

// 防止我们在上传代码的时候，没有将env改成production
const envVersion = wx.getAccountInfoSync().miniProgram.envVersion
if (envVersion === "release" && env !== "production") {
  env = "production"
}

export default {
  // 当前环境
  env,
  // 请求接口域名
  baseUrl: {
    develop: 'http://localhost:8080',  // 自定义api
    production: 'https://lihailong.com:8081',
  },
}
```



> app.js：定义getConfig函数，调用该函数可以获取全局参数

```js
// app.js
import localConfigs from "./config"

App({
  // 获取配置
  getConfig(key = "") {
    // 不指定key，返回全部
    if (key === "") {
      return localConfigs
    }
    // 不存在配置
    if (!localConfigs[key]) {
      console.warn(`${key} config is no exist`)
      return undefined
    }
    // 配置是否区分环境
    if (typeof localConfigs[key] === "object" && typeof localConfigs[key] !== null) {
      // 获取当前环境类型
      const env = this.getConfig("env")
      return localConfigs[key][env]
    }
    return localConfigs[key]
  }
})
```



## 自定义封装Promise风格请求

Promise风格的请求能够解决**回调地狱**的问题。

同时我们可以在自定义请求中携带部分全局信息（比如header请求中携带token）。

封装的请求参数如下：

- url 请求url
- method 请求方式，默认`GET`
- data 请求数据
- timeout 请求最长时间(ms)，默认20000，即20s
- showLoading 是否展示loading，默认为`true`

> util/request.js：

```js
const app = getApp()
export function createRequest(options) {
  return new Promise((resolve, reject) => {
    const token = wx.getStorageSync('token')  // 自定义：获取token

    const baseUrl = app.getConfig("baseUrl")
    const url = `${baseUrl}${options.url}`
    const header = {  // 自定义header
      'token': token
    }
    let showLoading = false
    if (options.loading !== false) {
      showLoading = true
      wx.showLoading({
        title: '正在加载',
        mask: true
      })
    }
    wx.request({
      url,
      method: options.method || 'GET',  // 自定义，默认GET方法
      timeout: options.timeout || 20000,  // 自定义，默认等待20s
      header,
      data: options.data || {},
      success(res) {
        /* 自定义后端响应格式
        res.data的格式
        {
          code: 200,
          success: true,
          msg: '响应正常',
          data: {}
        }
        */
        res = res.data
        if (showLoading) {
          wx.hideLoading()
          showLoading = false
        }
        if (res.success) {
          return resolve(res)
        } else {
          wx.showToast({  // 自定义：如果执行失败，则弹框提醒
            title: res.msg,
            icon: 'none'
          })
          return reject(res)
        }
      },
      fail(err) {
        if (showLoading) {
          wx.hideLoading()
          showLoading = false
        }
        wx.showToast({  // 自定义：如果执行失败，则弹框提醒
          title: '服务开小差啦！',
          icon: 'none'
        })
        reject(err)
      }
    })
  })
}
```



> 接口的定义和调用

个人习惯将接口的定义和接口的调用分开写。

- 定义接口：api/index/sendRequest.js

```js
import { createRequest } from "../../util/request";

export function sendRequest(data) {  // 定义接口
  return createRequest({  // 返回一个Promise对象
    url: '/test',
    method: 'get', 
    data: data
  })
}
```

- 调用接口：pages/index/index.js

```js
import { sendRequest } from "../../api/index/sendRequest"

// index.js
Page({
  clickButton() {  // 随便定义了一个函数
    sendRequest({'username': 'lhl', 'password': '123'})  // 调用接口的时候，仅需要传参
    .then(resp => console.log(resp))
    .catch(err => console.log(err))
  }
})
```



## 自定义封装Promise风格上传

> util/request.js：

```js
export function createUploadRequest(options) {
  return new Promise((resolve, reject) => {
    const token = wx.getStorageSync('token')  // 自定义：获取token
    const baseUrl = app.getConfig("baseUrl")
    const url = `${baseUrl}${options.url}`
    const header = {  // 自定义header
      'token': token
    }
    let showLoading = false
    if (options.loading !== false) {
      showLoading = true
      wx.showLoading({
        title: '正在加载',
        mask: true
      })
    }

    // todo
    wx.uploadFile({
      url,
      filePath: options.data.filePath,
      header,
      name: 'file',
      formData: { user: 'test' },
      timeout: options.timeout || 20000,
      success: (res) => {
        res = JSON.parse(res.data)
        if (showLoading) {
          wx.hideLoading()
          showLoading = false
        }
        if (res.success) {
          return resolve(res)
        } else {
          wx.showToast({  // 自定义：如果执行失败，则弹框提醒
            title: res.msg,
            icon: 'none'
          })
          return reject(res)
        }
      },
      fail(err) {
        if (showLoading) {
          wx.hideLoading()
          showLoading = false
        }
        wx.showToast({  // 自定义：如果执行失败，则弹框提醒
          title: '服务开小差啦！',
          icon: 'none'
        })
        reject(err)
      }
    });
  })
}
```



定义接口

```js
import { createUploadRequest } from '../../../util/request'

export function uploadImage(data) {
  return createUploadRequest({
    url: '/upload/image',
    method: 'POST', 
    data: data
  })
}
```



调用接口

```js
import { uploadImage } from '../../../api/trade/create-page/create-page'
Page({
  afterRead(e) {
    const { file } = e.detail  // 取出文件的临时地址，至关重要！
    // 调用函数
    uploadImage({
      filePath: file.url,
    }).then(res => {
      // 上传完成需要更新 fileList
      const { fileList = [] } = this.data;
      fileList.push({ ...file, url: res.data.url });
      this.setData({ fileList });
    })
  },
})

```

