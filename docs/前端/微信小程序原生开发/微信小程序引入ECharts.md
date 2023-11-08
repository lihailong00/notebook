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
<!-- 
1. 必须指定ec-canvas的长度和宽度
2. 必须指定ec-canvas标签的ec属性
3. 建议制定id属性和canvas-id属性
-->
<view style="width: 100%; height: 500rpx;">
  <ec-canvas id="grade-pie-cart" canvas-id="grade-pie-cart" ec="{{grade_pie_cart_ec}}" style="width: 100%; height: 100%;"></ec-canvas>
</view>
```



> JS

```js
import * as echarts from '../../ec-canvas/echarts'  // 引入官方的echarts组件

Page({
  data: {
    grade_pie_cart_ec: {  // 自定义ec变量
      onInit: initChart
    }
  }
});

function initChart(canvas, width, height, dpr) {
  let chart = echarts.init(canvas, null, {
    width: width,
    height: height,
    devicePixelRatio: dpr
  });
  canvas.setChart(chart);

  let option = getOption()

  chart.setOption(option);
  return chart;
}

function getOption() {
  let option = {  // 官网上查看参数案例：https://echarts.apache.org/examples/zh/index.html
    title: {
      text: '成绩分布',
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
          { value: 1048, name: '90-100' },
          { value: 735, name: '80-89' },
          { value: 580, name: '70-79' },
          { value: 484, name: '60-69' },
          { value: 300, name: '0-59' }
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
  };
  return option
}
```



> JSON

```json
{
  "usingComponents": {
    "ec-canvas": "/ec-canvas/ec-canvas"
  }
}
```

