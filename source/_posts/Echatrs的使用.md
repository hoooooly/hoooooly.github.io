---
title: Echatrs的使用
tags:
  - javascript
  - echarts
comments: true
typora-root-url: Echatrs的使用
date: 2021-07-07 16:57:13
categories:
index_img:
banner_img:
---

## 快速上手

### 获取 ECharts

你可以通过以下几种方式获取 `Apache EChartsTM`。

- 从`Apache ECharts`官网下载界面 获取官方源码包后构建。
- 在`ECharts`的`GitHub`获取。
- 通过 `npm` 获取` echarts，npm install echarts --save`，详见“在 `webpack `中使用` echarts`”
- 通过 `jsDelivr `等` CDN `引入

### 引入` ECharts`

通过标签方式直接引入构建好的 `echarts`文件

```html
<script src="https://cdn.bootcdn.net/ajax/libs/echarts/5.1.2/echarts.common.js"></script>
```

### 绘制一个简单的图表

在绘图前我们需要为 `ECharts` 准备一个具备高宽的 `DOM` 容器。

```html
<div id="line" style="width: 60%;height:400px;margin-left: 20%;margin-right:20%;margin-bottom: 20px"></div>
```

然后就可以通过 `echarts.init` 方法初始化一个 `echarts` 实例并通过 `setOption` 方法生成一个简单的柱状图，下面是完整代码。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Echarts</title>
    <!--引入echarts.js-->
<!--    <script type="text/javascript" src="/static/echarts.js"></script>-->
    <script src="https://cdn.bootcdn.net/ajax/libs/echarts/5.1.2/echarts.common.js"></script>
</head>
<body>
<!-- 为ECharts准备一个具备大小（宽高）的Dom -->
<div id="line" style="width: 60%;height:400px;margin-left: 20%;margin-right:20%;margin-bottom: 20px"></div>
<script type="text/javascript">
    // 柱形图图
    // 基于准备好的dom，初始化echarts实例
    var BarChart = echarts.init(document.getElementById('bar'));

    // 指定图表的配置项和数据
    var bar_option = {
        title: {
            text: 'ECharts 入门柱形图示例'
        },
        tooltip: {},
        legend: {
            data: ['销量']
        },
        xAxis: {
            data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
        },
        yAxis: {},
        series: [{
            name: '销量',
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20]
        }]
    };
</script>
</body>
</html>
```


这样你的第一个图表就诞生了！

<script src="https://cdn.bootcdn.net/ajax/libs/echarts/5.1.2/echarts.common.js"></script>


<div id="bar" style="width: 80%;height:400px;margin-left: 10%;margin-right:10%;margin-bottom: 20px"></div>

### 折线图

```javascript
// 折线图
// 基于准备好的dom，初始化echarts实例
var LineChart = echarts.init(document.getElementById('line'));

// 指定图表的配置项和数据
var line_option = {
    title: {
        text: 'ECharts 入门折线图示例',
        subtext: '销量数据分析'
    },
    tooltip: {},
    legend: {
        data: ['袜子', '裤子', '衬衫', '平均销量']
    },
    xAxis: {
        data: ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期日"]
    },
    yAxis: {},
    series: [{
        name: '袜子',
        type: 'line',
        data: [220, 136, 101, 120, 290, 124, 235]
    },
        {
            name: '裤子',
            type: 'line',
            data: [52, 928, 220, 136, 101, 120, 290]
        },
        {
            name: '衬衫',
            type: 'line',
            data: [512, 220, 136, 101, 120, 290, 1123]
        },
        {
            name: '平均销量',
            type: 'bar',
            data: [200, 220, 136, 300, 120, 250, 500]
        }]
};

// 使用刚指定的配置项和数据显示图表。
LineChart.setOption(line_option);
```


<div id="line" style="width: 80%;height:400px;margin-left: 10%;margin-right:10%;margin-bottom: 20px"></div>

### 饼图

```javascript
// 饼图
    // 基于准备好的dom，初始化echarts实例
    var PieChart = echarts.init(document.getElementById('pie'));

    // 指定图表的配置项和数据
    var pie_option = {
        title: {
            text: 'ECharts 入门饼图示例1'
        },
        tooltip: {},
        legend: {
            data: ['直接访问', '邮件营销', '联盟广告', '视频广告', '搜索引擎']
        },
        series: [{
            type: 'pie',
            radius: '55%',
            center: ['50%', '50%'],
            data: [
                {name: '直接访问', value: 335},
                {name: '邮件营销', value: 720},
                {name: '联盟广告', value: 135},
                {name: '视频广告', value: 500},
                {name: '搜索引擎', value: 235},
            ],
            roseType: 'radius'
        }]
    };

    // 使用刚指定的配置项和数据显示图表。
    PieChart.setOption(pie_option);
```

<div id="pie" style="width: 95%;height:400px;margin-left: 2.5%;margin-right:2.5%;margin-bottom: 20px;"></div>

```javascript
    // 饼图
    // 基于准备好的dom，初始化echarts实例
    var Pie1Chart = echarts.init(document.getElementById('pie1'));

    // 指定图表的配置项和数据
    var pie1_option = {
        title: {
            text: 'ECharts 入门饼图示例2'
        },
        tooltip: {},
        legend: {
            data: ['直接访问', '邮件营销', '联盟广告', '视频广告', '搜索引擎']
        },
        series: [{
            type: 'pie',
            radius: ['55%', '70%'],
            label: {
                normal: {
                    // 取消原来的显示模式
                    show: false,
                    // 在中间显示
                    position: 'center'
                }
            },
            emphasis: {
                label: {
                    show: true,
                    fontSize: '40',
                    fontWeight: 'bold'
                }
            },
            // center: ['50%', '50%'],
            data: [
                {name: '直接访问', value: 335},
                {name: '邮件营销', value: 720},
                {name: '联盟广告', value: 135},
                {name: '视频广告', value: 500},
                {name: '搜索引擎', value: 235},
            ],
        }]
    };

    // 使用刚指定的配置项和数据显示图表。
    Pie1Chart.setOption(pie1_option);
```

<div id="pie1" style="width: 95%;height:400px;margin-left: 2.5%;margin-right:2.5%;margin-bottom: 20px;"></div>

<script type="text/javascript" >
    // 柱形图图
    // 基于准备好的dom，初始化echarts实例
    var BarChart = echarts.init(document.getElementById('bar'));

    // 指定图表的配置项和数据
    var bar_option = {
        title: {
            text: 'ECharts 入门柱形图示例'
        },
        tooltip: {},
        legend: {
            data: ['销量']
        },
        xAxis: {
            data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
        },
        yAxis: {},
        series: [{
            name: '销量',
            type: 'bar',
            data: [5, 20, 36, 10, 10, 20]
        }]
    };
    
    // 使用刚指定的配置项和数据显示图表。
    BarChart.setOption(bar_option);
    
    // 折线图
    // 基于准备好的dom，初始化echarts实例
    var LineChart = echarts.init(document.getElementById('line'));
    
    // 指定图表的配置项和数据
    var line_option = {
        title: {
            text: 'ECharts 入门折线图示例',
            subtext: '销量数据分析'
        },
        tooltip: {},
        legend: {
            data: ['袜子', '裤子', '衬衫', '平均销量']
        },
        xAxis: {
            data: ["星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期日"]
        },
        yAxis: {},
        series: [{
            name: '袜子',
            type: 'line',
            data: [220, 136, 101, 120, 290, 124, 235]
        },
            {
                name: '裤子',
                type: 'line',
                data: [52, 928, 220, 136, 101, 120, 290]
            },
            {
                name: '衬衫',
                type: 'line',
                data: [512, 220, 136, 101, 120, 290, 1123]
            },
            {
                name: '平均销量',
                type: 'bar',
                data: [200, 220, 136, 300, 120, 250, 500]
            }]
    };
    
    // 使用刚指定的配置项和数据显示图表。
    LineChart.setOption(line_option);


    // 饼图
    // 基于准备好的dom，初始化echarts实例
    var PieChart = echarts.init(document.getElementById('pie'));
    
    // 指定图表的配置项和数据
    var pie_option = {
        title: {
            text: 'ECharts 入门饼图示例1'
        },
        tooltip: {},
        legend: {
            data: ['直接访问', '邮件营销', '联盟广告', '视频广告', '搜索引擎']
        },
        series: [{
            type: 'pie',
            radius: '55%',
            center: ['50%', '50%'],
            data: [
                {name: '直接访问', value: 335},
                {name: '邮件营销', value: 720},
                {name: '联盟广告', value: 135},
                {name: '视频广告', value: 500},
                {name: '搜索引擎', value: 235},
            ],
            roseType: 'radius'
        }]
    };
    
    // 使用刚指定的配置项和数据显示图表。
    PieChart.setOption(pie_option);


    // 饼图
    // 基于准备好的dom，初始化echarts实例
    var Pie1Chart = echarts.init(document.getElementById('pie1'));
    
    // 指定图表的配置项和数据
    var pie1_option = {
        title: {
            text: 'ECharts 入门饼图示例2'
        },
        tooltip: {},
        legend: {
            data: ['直接访问', '邮件营销', '联盟广告', '视频广告', '搜索引擎']
        },
        series: [{
            type: 'pie',
            radius: ['55%', '70%'],
            label: {
                normal: {
                    // 取消原来的显示模式
                    show: false,
                    // 在中间显示
                    position: 'center'
                }
            },
            emphasis: {
                label: {
                    show: true,
                    fontSize: '40',
                    fontWeight: 'bold'
                }
            },
            // center: ['50%', '50%'],
            data: [
                {name: '直接访问', value: 335},
                {name: '邮件营销', value: 720},
                {name: '联盟广告', value: 135},
                {name: '视频广告', value: 500},
                {name: '搜索引擎', value: 235},
            ],
        }]
    };
    
    // 使用刚指定的配置项和数据显示图表。
    Pie1Chart.setOption(pie1_option);
</script>










[//]:#(设置表格整体居中显示)
<style>
    table
    {
        margin: auto;
        font-size: 80%;
    }
</style>


