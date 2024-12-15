### echarts使用

[echarts官网配置项教程](https://echarts.apache.org/zh/option.html#title)



#### 1.堆叠图

https://echarts.apache.org/examples/zh/editor.html?c=bar-brush



series中`stack`属性值保持一致，就在堆叠在一起

```ts
 option = {
  title: {
    text: 'Stacked Line'
  },
  // tooltip 鼠标悬浮时， 提示框组件显示
  tooltip: {
    trigger: 'axis'
  },
  // 图标侧边显示的图例
  legend: {
    type: 'scroll', // 'plain'：普通图例。 'scroll'：可滚动翻页的图例。当图例数量较多时可以使用。
    orient: 'vertical', // 图例列表的布局朝向。 'horizontal'： 横向，'vertical'：垂直  默认horizontal
    data: ['Email', 'Union Ads', 'Video Ads', 'Direct', 'Search Engine']
  },
  // 显示图表， 多个图，就有多个grid,
  grid: {
    left: '3%',
    right: '4%',
    bottom: '3%',
    containLabel: true
  },
  // 工具栏
  toolbox: {
    feature: {
       magicType: {
        type: ['stack']
      },
      saveAsImage: {}
    }
  },
   // 坐标系grid中的x轴
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
  },
  // 坐标系grid中的y轴
  yAxis: {
    type: 'value'
  },
  series: [
    {
      name: 'Email',
      type: 'bar',
      stack: 'Total',
      data: [120, 132, 101, 134, 90, 230, 210]
    },
    {
      name: 'Union Ads',
      type: 'bar',
      stack: 'Total',
      data: [220, 182, 191, 234, 290, 330, 310]
    },
    {
      name: 'Video Ads',
      type: 'bar',
      stack: 'Total',
      data: [150, 232, 201, 154, 190, 330, 410]
    },
    {
      name: 'Direct',
      type: 'bar',
      stack: 'Total',
      data: [320, 332, 301, 334, 390, 330, 320]
    },
    {
      name: 'Search Engine',
      type: 'bar',
      stack: 'Total',
      data: [820, 932, 901, 934, 1290, 1330, 1320]
    }
  ]
};
```

#### 2.多图，共用同一个x轴

```
option = {
  tooltip: {
    trigger: 'axis',
    axisPointer: {
      // 坐标轴指示器，坐标轴触发有效
      type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
    }
  },
  legend: {
    data: ['水位', '降雨', '流量']
  },
  //多个图，就有多个grid,排序默认0、1、2....
  grid: [
    //0降雨
    { x: '8%', y: '7%', height: '38%' },
    //1水位流量
    { x: '8%', y2: '7%', height: '46%' }
  ],
  //多个图，就要定义多个x轴，虽然我这个表的x轴是一样的
  xAxis: [
    {
      show: false, //隐藏了x轴
      type: 'category',
      gridIndex: 0, //对应前面grid的索引位置（第一个）
      axisTick: {
        alignWithLabel: true
      },
      // axisLabel:{
      //         interval:2,  //x轴显示的数量，我这里是动态算的
      //     },
      data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
    },
    {
      type: 'category',
      gridIndex: 1, //对应前面grid的索引位置（第二个）
      axisTick: {
        alignWithLabel: true
      },
      // axisLabel:{
      //         interval:showNum,
      //     },
      data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
    }
  ],
  //y轴，不管有几个x轴，几个y轴，或者图，只要找到他对应的grid图的序号索引就可以精准匹配
  yAxis: [
    {
      type: 'value',
      gridIndex: 0, //对应前面grid的索引位置（第二个）
      name: '降雨',
      nameLocation: 'end',
      // min: swMin,
      // max: swMax,
      position: 'right',
      axisLine: {
        lineStyle: {
          color: '#5793f3'
        }
      },
      axisLabel: {
        formatter: '{value}'
      }
    },
    {
      type: 'value',
      gridIndex: 1,
      name: '水位 m',
      nameLocation: 'end',
      // min: swMin,
      // max: swMax,
      position: 'right',
      axisLine: {
        lineStyle: {
          color: '#5793f3'
        }
      },
      axisLabel: {
        formatter: '{value}'
      }
    }
  ],
  series: [
    {
      name: 'Evaporation',
      type: 'bar',
      data: [
        2.0, 4.9, 7.0, 23.2, 25.6, 76.7, 135.6, 162.2, 32.6, 20.0, 6.4, 3.3
      ]
    },
    {
      name: 'Evaporation',
      type: 'bar',
      xAxisIndex: 1,
      yAxisIndex: 1,
      data: [
        2.0, 4.9, 7.0, 23.2, 25.6, 76.7, 135.6, 162.2, 32.6, 20.0, 6.4, 3.3
      ]
    }
  ]
};

                 
```

#### 3.多y轴，共同一个x轴

```
const colors = ['#5470C6', '#91CC75', '#EE6666'];
option = {
  color: colors,
  tooltip: {
    trigger: 'axis',
    axisPointer: { // 坐标轴指示器，坐标轴触发有效
      type: 'shadow' //  /默认为直线，可选为：'line' | 'shadow' | 'cross'， line为竖线， shadow为柱状背景色，cross为十字架虚线
    }
  },
   //多个图，就有多个grid,排序默认0、1、2....
  grid: {
    right: '20%'
  },
  toolbox: {
    feature: {
      dataView: { show: true, readOnly: false },
      restore: { show: true },
      saveAsImage: { show: true }
    }
  },
  legend: {
    data: ['Evaporation', 'Precipitation', 'Temperature']
  },
  xAxis: [
    {
      type: 'category',
      axisTick: {
        alignWithLabel: true
      },
      // prettier-ignore
      data: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
    }
  ],
  yAxis: [
    {
      type: 'value',
      name: 'Evaporation',
      position: 'left',
      alignTicks: true,
      axisLine: {
        show: true,
        lineStyle: {
          color: colors[0]
        }
      },
      axisLabel: {
        formatter: '{value} ml'
      }
    },
    {
      type: 'value',
      name: 'Precipitation',
      position: 'right',
      alignTicks: true,
      axisLine: {
        show: true,
        lineStyle: {
          color: colors[1]
        }
      },
      axisLabel: {
        formatter: '{value} ml'
      }
    },
    {
      type: 'value',
      name: '温度',
      position: 'right',
      offset: 80,
      alignTicks: true,
      axisLine: {
        show: true,
        lineStyle: {
          color: colors[2]
        }
      },
      axisLabel: {
        formatter: '{value} °C'
      }
    }
  ],
  series: [
    {
      name: 'Evaporation',
      type: 'bar',
      data: [
        2.0, 4.9, 7.0, 23.2, 25.6, 76.7, 135.6, 162.2, 32.6, 20.0, 6.4, 3.3
      ]
    },
    {
      name: 'Precipitation',
      type: 'bar',
      yAxisIndex: 1,
      data: [
        2.6, 5.9, 9.0, 26.4, 28.7, 70.7, 175.6, 182.2, 48.7, 18.8, 6.0, 2.3
      ]
    },
    {
      name: 'Temperature',
      type: 'line',
      yAxisIndex: 2,
      data: [2.0, 2.2, 3.3, 4.5, 6.3, 10.2, 20.3, 23.4, 23.0, 16.5, 12.0, 6.2]
    }
  ]
};
```
