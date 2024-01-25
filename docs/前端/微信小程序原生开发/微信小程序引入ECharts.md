# 微信小程序引入ECharts

[toc]



## 第一步：下载echarts-for-weixin

Github地址：https://github.com/ecomfe/echarts-for-weixin



## 第二步：打开官方echarts项目

将根目录的`ec-canvas`目录拷贝到我们项目的根目录下。



## 第三步：使用echarts

可以借鉴官方项目中使用echarts的案例，也可以参考下面的案例。



> HTML

```html
<navigation-bar title="lazy echart" back="{{false}}" color="black" background="#FFF"></navigation-bar>
<!-- 
1. 必须指定ec-canvas的长度和宽度
2. 必须指定ec-canvas标签的ec属性
3. 建议制定id属性和canvas-id属性
-->

<!-- 图1 -->
<view class="my-chart" style="width: 100%; height: 1000rpx;">
  <ec-canvas id="graph1" canvas-id="chart1" ec="{{ec1}}" style="width: 100%; height: 100%;"></ec-canvas>
</view>

<!-- 图2 -->
<view class="my-chart" style="width: 100%; height: 1000rpx;">
  <ec-canvas id="graph2" canvas-id="chart2" ec="{{ec2}}" style="width: 100%; height: 100%;"></ec-canvas>
</view>
```



> JS

```js
// pages/lazy-echart/lazy-echart.js
import * as echarts from '../../ec-canvas/echarts'

Page({

  /**
   * 页面的初始数据
   */
  data: {
    ec1: {
      lazyLoad: true
    },
    ec2: {
      lazyLoad: true
    }
  },
  onLoad(options) {
    this.component1 = this.selectComponent("#graph1")  // 获取图1dom
    this.component2 = this.selectComponent("#graph2")  // 获取图2dom
    this.init()
  },
  init() {
    // 图1初始化
    this.component1.init((canvas, width, height, dpr) => {
      let chart = echarts.init(canvas, null, {
        width,
        height,
        devicePixelRatio: dpr
      })
      let option = getOption1()
      chart.setOption(option)
      this.chart = chart
      return chart
    })

    // 图2初始化
    this.component2.init((canvas, width, height, dpr) => {
      let chart = echarts.init(canvas, null, {
        width,
        height,
        devicePixelRatio: dpr
      })
      let option = getOption2()
      chart.setOption(option)
      this.chart = chart
      return chart
    })
  }
})

function getOption1() {  // 获取图1配置信息
  wx.request({
    url: 'http://localhost:8080/graph/1',  // 自定义后端获取数据
  })
  
  let option = {
    title: {
      text: 'Referer of a Website',
      subtext: 'Fake Data',
      left: 'center'
    },
    xAxis: {
      type: 'category',
      data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
      type: 'value'
    },
    series: [
      {
        data: [150, 230, 224, 218, 135, 147, 260],
        type: 'line'
      }
    ]
  }

  option.title.text = '第一个图'  // 自定义参数
  return option
}

function getOption2() {  // 获取图2配置信息
  wx.request({
    url: 'http://localhost:8080/graph/2',  // 自定义后端获取数据
  })
  
  let option = {
    title: {
      text: 'Referer of a Website',
      subtext: 'Fake Data',
      left: 'center'
    },
    tooltip: {
      trigger: 'item'
    },
    legend: {
      orient: 'vertical',
      left: 'left'
    },
    series: [
      {
        name: 'Access From',
        type: 'pie',
        radius: '50%',
        data: [
          { value: 1048, name: 'Search Engine' },
          { value: 735, name: 'Direct' },
          { value: 580, name: 'Email' },
          { value: 484, name: 'Union Ads' },
          { value: 300, name: 'Video Ads' }
        ],
        emphasis: {
          itemStyle: {
            shadowBlur: 10,
            shadowOffsetX: 0,
            shadowColor: 'rgba(0, 0, 0, 0.5)'
          }
        }
      }
    ]
  }

  option.title.text = '第二个图'  // 自定义参数
  return option
}
```



> JSON

```json
{
  "usingComponents": {
    "navigation-bar": "/components/navigation-bar/navigation-bar",
    "ec-canvas": "../../ec-canvas/ec-canvas"
  }
}
```

