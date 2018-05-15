---
title: ECharts GL 实现常见三维可视化
date: 2018-05-15 20:31:15
author: techotter
tags: 
  - ECharts
  - 三维可视化
  - 数据可视化
categories:
  - 数据可视化
---

ECharts GL（后面统一简称 GL）是 ECharts 的三维可视化扩展，为 ECharts 补充了丰富的三维可视化组件。本文将简单介绍如何基于 GL 实现一些常见的三维可视化作品。如果你对 ECharts 有一定了解的话，可以很快上手 GL，因为 GL 的配置项完全是按照 ECharts 的标准和上手难度来设计的。通过本文，你可以对 GL 的基本使用有一个大致的了解，之后可以前往 [官方示例](https://www.echartsjs.com/examples/zh/index.html#chart-type-globe) 和 [Gallery](https://gallery.echartsjs.com/explore.html#sort=rank~timeframe=all~author=all) 去查看更多使用 GL 制作的示例。对于文中没有详细解释的代码，也可以查阅 [GL 配置项手册](https://www.echartsjs.com/zh/option-gl.html) 了解具体的配置项使用方法。

<!-- more -->

## 如何下载和引入 ECharts GL

为了不再增加已经很大了的 ECharts 完整版的体积，GL 作为扩展包的形式提供，与诸如水球图之类的扩展类似。如果要使用 GL 的各种组件，只需要在引入 `echarts.min.js` 的基础上再引入一个 `echarts-gl.min.js`。

你可以从 [官网](https://www.echartsjs.com/zh/download.html) 下载最新版的 GL，然后在页面中通过标签引入：

```html
<script src="lib/echarts.min.js"></script>
<script src="lib/echarts-gl.min.js"></script>
```

如果你的项目使用 webpack 或者 rollup 来打包代码，也可以通过 npm 安装后引入：

```bash
npm install echarts
npm install echarts-gl
```

然后通过 ES6 的 import 语法引入 ECharts 和 ECharts GL：

```javascript
import echarts from 'echarts';
import 'echarts-gl';
```

## 声明一个基础的三维笛卡尔坐标系

引入 ECharts 和 ECharts GL 后，我们先来声明一个基础的三维笛卡尔坐标系，用于绘制三维的散点图、柱状图、曲面图等常见的统计图。

在 ECharts 中，我们使用 `grid` 组件来提供一个矩形区域，用于放置一个二维的笛卡尔坐标系，以及笛卡尔坐标系上的 x 轴（`xAxis`）和 y 轴（`yAxis`）。对于三维的笛卡尔坐标系，GL 提供了 `grid3D` 组件，用于划分一块三维的笛卡尔空间，并在这个 `grid3D` 上放置 `xAxis3D`、`yAxis3D` 和 `zAxis3D`。

> 小提示：在 GL 中，除了 `globe` 之外，所有的三维组件和系列都加了 `3D` 后缀以示区分，例如三维的散点图是 `scatter3D`，三维的地图是 `map3D` 等等。

下面的代码声明了一个最简单的三维笛卡尔坐标系：

```javascript
let option = {
    // 需要注意的是我们不能跟 grid 一样省略 grid3D
    grid3D: {},
    // 默认情况下, x, y, z 分别是从 0 到 1 的数值轴
    xAxis3D: {},
    yAxis3D: {},
    zAxis3D: {}
};
```


## 绘制三维散点图

声明好笛卡尔坐标系后，我们先试试用一份程序生成的正态分布数据在这个三维的笛卡尔坐标系中画散点图。

下面是生成正态分布数据的代码，你可以先不用关心这段代码的具体实现，只需要知道它生成了一份三维的正态分布数据，存放在 `data` 数组中：```javascript
function makeGaussian(amplitude, x0, y0, sigmaX, sigmaY) {
    return function (amplitude, x0, y0, sigmaX, sigmaY, x, y) {
        let exponent = -(
            (Math.pow(x - x0, 2) / (2 * Math.pow(sigmaX, 2))) +
            (Math.pow(y - y0, 2) / (2 * Math.pow(sigmaY, 2)))
        );
        return amplitude * Math.pow(Math.E, exponent);
    }.bind(null, amplitude, x0, y0, sigmaX, sigmaY);
}

// 创建一个高斯分布函数
const gaussian = makeGaussian(50, 0, 0, 20, 20);

let data = [];
for (let i = 0; i < 1000; i++) {
    // x, y 在 [-50, 50) 区间内随机分布
    let x = Math.random() * 100 - 50;
    let y = Math.random() * 100 - 50;
    let z = gaussian(x, y);
    data.push([x, y, z]);
}```

生成的正态分布数据大概长这样：```javascript
[
    [46.74395071259907, -33.88391024738553, 0.7754030099768191],
    [-18.45302873809771, 16.88114775416834, 22.87772504105404],
    [2.9908128281121336, -0.027699444453467947, 49.44400635911886],
    ...
]```

每一项都包含了 x、y、z 三个值，这三个值会分别被映射到笛卡尔坐标系的 x 轴、y 轴和 z 轴上。然后我们可以使用 GL 提供的 `scatter3D` 系列类型把这些数据画成三维空间中正态分布的点：```javascript
let option = {
    grid3D: {},
    xAxis3D: {},
    yAxis3D: {},
    zAxis3D: { max: 100 },
    series: [{
        type: 'scatter3D',
        data: data
    }]
};```


## 使用真实数据的三维散点图

接下来我们来看一个使用真实多维数据的三维散点图例子。开始之前可以先从 [这里](https://www.echartsjs.com/examples/data/asset/data/life-expectancy-table.json) 获取这份数据。

编辑器里格式化一下可以看到这份数据是很传统的转成 JSON 后的表格格式。第一行是每一列数据的属性名，可以从属性名看出每一列数据的含义，分别是人均收入、人均寿命、人口数量、国家和年份：```javascript
[
    ["Income", "Life Expectancy", "Population", "Country", "Year"],
    [815, 34.05, 351014, "Australia", 1800],
    [1314, 39, 645526, "Canada", 1800],
    [985, 32, 321675013, "China", 1800],
    [864, 32.2, 345043, "Cuba", 1800],
    [1244, 36.5731262, 977662, "Finland", 1800],
    ...
]```

在 ECharts 4 中我们可以使用 `dataset` 组件非常方便地引入这份数据。如果对 `dataset` 还不熟悉的话可以看 [dataset 使用教程](https://www.echartsjs.com/zh/tutorial.html#%E4%BD%BF%E7%94%A8%20dataset%20%E7%AE%A1%E7%90%86%E6%95%B0%E6%8D%AE)：```javascript
$.getJSON('data/asset/data/life-expectancy-table.json', function (data) {
    myChart.setOption({
        grid3D: {},
        xAxis3D: {},
        yAxis3D: {},
        zAxis3D: {},
        dataset: {
            source: data
        },
        series: [
            {
                type: 'scatter3D',
                symbolSize: 2.5
            }
        ]
    });
});```

ECharts 默认会把前三列，也就是收入（Income）、人均寿命（Life Expectancy）、人口（Population）分别放到 x、y、z 轴上。

使用 `encode` 属性我们还可以将指定列的数据映射到指定的坐标轴上，从而省去很多繁琐的数据转换代码。例如我们将 x 轴换成国家（Country），y 轴换成年份（Year），z 轴换成收入（Income），可以看到不同国家不同年份的人均收入分布：```javascript
myChart.setOption({
    grid3D: {},
    xAxis3D: {
        // 因为 x 轴和 y 轴都是类目数据，所以需要设置 type: 'category' 保证正确显示数据。
        type: 'category'
    },
    yAxis3D: {
        type: 'category'
    },
    zAxis3D: {},
    dataset: {
        source: data
    },
    series: [
        {
            type: 'scatter3D',
            symbolSize: 2.5,
            encode: {
                // 维度的名字默认就是表头的属性名
                x: 'Country',
                y: 'Year',
                z: 'Income',
                tooltip: [0, 1, 2, 3, 4]
            }
        }
    ]
});```


## 利用 visualMap 组件对三维散点图进行视觉编码

在刚才多维数据的例子中，我们还有几个维度（列）没能表达出来。利用 ECharts 内置的 `visualMap` 组件，我们可以继续将第四个维度编码成颜色：```javascript
myChart.setOption({
    grid3D: {
        viewControl: {
            // 使用正交投影。
            projection: 'orthographic'
        }
    },
    xAxis3D: {
        // 因为 x 轴和 y 轴都是类目数据，所以需要设置 type: 'category' 保证正确显示数据。
        type: 'category'
    },
    yAxis3D: {
        type: 'log'
    },
    zAxis3D: {},
    visualMap: {
        calculable: true,
        max: 100,
        // 维度的名字默认就是表头的属性名
        dimension: 'Life Expectancy',
        inRange: {
            color: ['#313695', '#4575b4', '#74add1', '#abd9e9', '#e0f3f8', '#ffffbf', '#fee090', '#fdae61', '#f46d43', '#d73027', '#a50026']
        }
    },
    dataset: {
        source: data
    },
    series: [
        {
            type: 'scatter3D',
            symbolSize: 5,
            encode: {
                // 维度的名字默认就是表头的属性名
                x: 'Country',
                y: 'Population',
                z: 'Income',
                tooltip: [0, 1, 2, 3, 4]
            }
        }
    ]
});
```

这段代码中我们又在刚才的例子基础上加入了 `visualMap` 组件，将 Life Expectancy 这一列数据映射到了不同的颜色。除此之外，我们还把原来默认的透视投影改成了正交投影。正交投影在某些场景中可以避免因为近大远小所造成的表达错误。

除了 `visualMap` 组件，还可以利用其它的 ECharts 内置组件，并充分利用这些组件的交互效果，比如 `legend`。也可以像 [三维散点图和散点矩阵结合使用](https://www.echartsjs.com/gallery/editor.html?c=scatter3d-scatter&gl=1) 这个例子一样，实现二维和三维的系列混搭。在实现 GL 的时候，我们尽可能地把 WebGL 和 Canvas 之间的差异降到最低，从而让 GL 的使用可以更加方便自然。


## 在笛卡尔坐标系上显示其它类型的三维图表

除了散点图，我们也可以通过 GL 在三维的笛卡尔坐标系上绘制其它类型的三维图表。比如刚才例子中将 `scatter3D` 类型改成 `bar3D`，就可以变成一个三维的柱状图。

还有机器学习中会用到的三维曲面图 `surface`，常用来表达平面上的数据走势。刚才的正态分布数据我们也可以像下面这样画成曲面图：

```javascript
let data = [];
// 曲面图要求给入的数据是网格形式按顺序分布。
for (let y = -50; y <= 50; y++) {
    for (let x = -50; x <= 50; x++) {
        let z = gaussian(x, y);
        data.push([x, y, z]);
    }
}

option = {
    grid3D: {},
    xAxis3D: {},
    yAxis3D: {},
    zAxis3D: { max: 60 },
    series: [{
        type: 'surface',
        data: data
    }]
};
```


除了上述提到的这些系列类型，GL 还提供了三维的 [line3D](https://www.echartsjs.com/gallery/editor.html?c=line3d-orthographic&gl=1)、[lines3D](https://www.echartsjs.com/gallery/editor.html?c=lines3d-airline-on-globe&gl=1)、[scatterGL](https://www.echartsjs.com/gallery/editor.html?c=scatterGL-gps&gl=1) 等更多系列类型。你可以前往 [官方示例](https://www.echartsjs.com/examples/zh/index.html#chart-type-globe) 和 [配置项手册](https://www.echartsjs.com/zh/option-gl.html) 查看更多信息。

## 绘制三维地图

除了笛卡尔坐标系，GL 还内置了 [地理坐标系（geo3D）](https://www.echartsjs.com/zh/option-gl.html#geo3D) 和 [地球（globe）](https://www.echartsjs.com/zh/option-gl.html#globe)。利用他们可以绘制出三维的地图。

在 ECharts 中绘制基础的平面地图时，我们一般会利用 [geo](https://www.echartsjs.com/zh/option.html#geo) 组件。在 GL 中也可以用 `geo3D` 组件绘制三维的地图。跟 `globe` 组件相比，`geo3D` 组件显示的只是平面地图的微微倾斜效果，并不是真正意义上的三维地球。但是 `geo3D` 组件同样支持 [map3D](https://www.echartsjs.com/zh/option-gl.html#series-map3D) 系列，可以在地图上显示三维的柱状图等其它图表。

下面是一个在三维地图上显示人口分布的示例：```javascript
$.getJSON('data/asset/data/population.json', function (data) {
    data = data
        .filter(function (dataItem) {
            return dataItem[2] > 0;
        })
        .map(function (dataItem) {
            return [dataItem[0], dataItem[1], Math.sqrt(dataItem[2])];
        });

    myChart.setOption({
        geo3D: {
            map: 'world',
            shading: 'lambert',
            light: {
                ambient: {
                    intensity: 0.4
                },
                main: {
                    intensity: 0.3
                }
            },
            viewControl: {
                distance: 50
            },
            groundPlane: {
                show: false
            },
            postEffect: {
                enable: true,
                SSAO: {
                    enable: true,
                    quality: 'high'
                }
            },
            temporalSuperSampling: {
                enable: true
            }
        },
        series: [{
            type: 'bar3D',
            coordinateSystem: 'geo3D',
            shading: 'lambert',
            data: data,
            barSize: 0.1,
            minHeight: 0.2,
            silent: true,
            itemStyle: {
                color: 'orange'
                // opacity: 0.8
            }
        }]
    });
});
```

真正意义上的三维地球要使用 [globe](https://www.echartsjs.com/zh/option-gl.html#globe) 组件。`globe` 组件提供了丰富的配置选项，除了常见的三维地图，还可以使用 [bar3D](https://www.echartsjs.com/gallery/editor.html?c=bar3d-on-globe&gl=1)、[scatter3D](https://www.echartsjs.com/gallery/editor.html?c=scatter3d-on-globe&gl=1) 等系列在地球表面或地球内部显示三维的图表。

如果要显示三维地球表面的飞线图，可以使用 [lines3D](https://www.echartsjs.com/gallery/editor.html?c=lines3d-airline-on-globe&gl=1) 系列。但是如果要让线条贴合在地球表面，就需要使用 GL 特有的 [linesGL](https://www.echartsjs.com/gallery/editor.html?c=linesGL-airline&gl=1) 系列了。下面这个例子是使用 `linesGL` 显示了世界飞机航线：```javascript
$.getJSON('data/asset/data/flights.json', function(data) {
    myChart.setOption({
        backgroundColor: '#000',
        globe: {
            baseTexture: 'data-gl/asset/world.topo.bathy.200401.jpg',
            heightTexture: 'data-gl/asset/world.topo.bathy.200401.jpg',
            shading: 'realistic',
            light: {
                ambient: {
                    intensity: 0.4
                },
                main: {
                    intensity: 0.4
                }
            },
            viewControl: {
                autoRotate: false
            }
        },
        series: [{
            type: 'linesGL',
            coordinateSystem: 'globe',
            blendMode: 'lighter',
            lineStyle: {
                color: 'rgb(50, 50, 150)',
                opacity: 0.1
            },
            data: data
        }]
    });
});
```

## 总结

本文对 ECharts GL 的基本使用方法进行了介绍，包括如何引入 GL、如何创建笛卡尔坐标系和地理坐标系、以及如何在这些坐标系中使用不同的三维系列。
