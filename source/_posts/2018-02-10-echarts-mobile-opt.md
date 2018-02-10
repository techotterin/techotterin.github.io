---
title: ECharts 在移动端的优化与自适应方案
date: 2018-02-10 10:30:05
author: techotter
tags:
  - ECharts
  - 移动端
  - 自适应
categories:
  - 数据可视化
---


# ECharts 在移动端的优化与自适应方案

在当今社会，我们越来越多地使用手机、平板等移动设备进行办公和获取数据。因此，图表在移动端的显示效果变得尤为重要。然而，由于移动设备屏幕尺寸有限，图表中的数据过多时，常常会出现遮挡和重叠的问题。为了解决这个问题，我们需要对移动端的图表显示进行优化。ECharts 对于移动端的优化和支持主要有两个方面。

## 一、ECharts 组件的定位和布局

ECharts 官方文档对组件的定位描述得比较详细和全面。简而言之，ECharts 对于图表中的每个组件和工具都采用了两种尺寸单位和固定位置的设置方式。

第一种是直接使用像素（px）进行设置，以 number 类型填写。例如：

```javascript
title: {
    text: 'ECharts 数据统计',
    top: 20
}
```

这里设置标题组件距离顶部的高度为 20px。

第二种是使用百分比（%）的形式进行设置，百分比值为 string 类型，需要加上引号。例如：

```javascript
legend: {
    data: ['访问量', '用户量'],
    left: '50%'
}
```

这里表示 legend 组件距离左侧的距离是整个图表宽度的 50%。

此外，还可以通过固定值来设置组件所在的位置，例如：

- 可以设置 `left: 'center'`，表示水平居中。
- 可以设置 `top: 'middle'`，表示垂直居中。

不同类型的图表还有其特有的定位方式。

布局方面可以简单归结为两种：横向（horizontal）显示和纵向（vertical）显示。

## 二、ECharts 的自适应能力 Media Query

Media Query 提供了随着容器尺寸改变而改变的能力。示例代码如下：

```javascript
option = {
    baseOption: {
        title: {...},
        legend: {...},
        series: [{...}, {...}, ...],
        ...
    },
    media: [
        {
            query: {...},
            option: {
                legend: {...},
                ...
            }
        },
        {
            query: {...},
            option: {
                legend: {...},
                ...
            }
        },
        {
            option: {
                legend: {...},
                ...
            }
        }
    ]
};
```

上面的例子中，`baseOption` 以及 `media` 中的每个 `option` 都是"原子 option"，即普通的含有各组件、系列定义的 option。而由"原子 option"组合成的整个 option，称为"复合 option"。`baseOption` 是必然被使用的，此外，满足某个 `query` 条件时，对应的 `option` 会被使用 `chart.mergeOption()` 方法进行合并。

需要注意的是，可以有多个 `query` 同时被满足，它们会按照定义的顺序依次被合并，后定义的优先级更高。

如果 `media` 中有某项没有写 `query`，则表示默认值，即所有规则都不满足时，采纳这个 option。

对于容器大小实时变化的情况，需要注意：如果某个配置项在某一个 `query` option 中出现，那么在其他 `query` option 中也必须出现，否则无法回归到原来的状态。（`left`/`right`/`top`/`bottom`/`width`/`height` 不受此限制。）

另外需要注意的是，"复合 option"中的 `media` 不支持 merge。也就是说，当多次调用 `chart.setOption(rawOption)` 时，如果 `rawOption` 是复合 option（包含 `media` 列表），那么新的 `rawOption.media` 列表不会和老的 `media` 列表进行合并，而是简单替代。当然，`rawOption.baseOption` 仍然会正常地与老的 option 进行合并。

实际上，很少有场景需要使用"复合 option"来多次调用 `setOption`。我们推荐的做法是，使用 `mediaQuery` 时，第一次 `setOption` 使用"复合 option"，后面的 `setOption` 仅使用"原子 option"，也就是仅仅用 `setOption` 来改变 `baseOption`。

## 三、另一种移动端自适应方案

除了使用 ECharts 提供的移动端自适应方法外，我还想提供另一种方案：通过 JavaScript 识别浏览器信息，然后根据所得信息设置图表容器的尺寸，再结合 ECharts 的 Media Query 来更好地展示图表。

检测是否为移动端的 JavaScript 代码如下：

```javascript
var isMobile = false;
var browser = {
    versions: function () {
        var u = navigator.userAgent, app = navigator.appVersion;
        return {
            trident: u.indexOf('Trident') > -1,
            presto: u.indexOf('Presto') > -1,
            webKit: u.indexOf('AppleWebKit') > -1,
            gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1,
            mobile: !!u.match(/AppleWebKit.*Mobile.*/) || !!u.match(/AppleWebKit/),
            ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/),
            android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1,
            iPhone: u.indexOf('iPhone') > -1 || u.indexOf('Mac') > -1,
            iPad: u.indexOf('iPad') > -1,
            webApp: u.indexOf('Safari') == -1
        };
    }(),
    language: (navigator.browserLanguage || navigator.language).toLowerCase()
};
isMobile = browser.versions.mobile;
```

这段代码能够识别大部分移动端设备的浏览器信息，但对于一些特殊的浏览器可能会存在缺陷。

接下来，根据浏览器尺寸设置图表容器的大小：

```javascript
if (browser.versions.mobile) {
    window.addEventListener("onorientationchange" in window ? "orientationchange" : "resize", hengshuping, false);
    $("#chartmain").height(pageheight * 0.6);
    $("#chartmain").width(pagewidth * 0.95);
} else {
    $("#chartmain").height("500px");
    $("#chartmain").width("700px");
}

function hengshuping() {
    if (window.orientation == 180 || window.orientation == 0) {
        $("#chartmain").height($(window).height() - 20);
        $("#chartmain").width("100%");
    }
    if (window.orientation == 90 || window.orientation == -90) {
        $("#chartmain").height($(window).height() - 20);
        $("#chartmain").width("100%");
    }
}
```

最后，结合 ECharts 的 Media Query 设置图表参数：

```javascript
function init() {
    var myChart = echarts.init(document.getElementById('chartmain'));
    option = {
        baseOption: {
            title: {
                text: '奶牛数字化养殖报表',
                subtext: '西部电子数据采集'
            },
            tooltip: {
                trigger: 'axis'
            },
            legend: {
                data: ['每日饲喂量', '产奶量']
            },
            toolbox: {
                show: true,
                feature: {
                    mark: { show: true },
                    dataView: { show: true, readOnly: false },
                    magicType: { show: true, type: ['line', 'bar', 'stack', 'tiled'] },
                    restore: { show: true },
                    saveAsImage: { show: true }
                }
            },
            calculable: true,
            xAxis: [
                {
                    type: 'category',
                    boundaryGap: false,
                    data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
                }
            ],
            yAxis: [
                {
                    type: 'value'
                }
            ],
            series: [
                {
                    name: '每日饲喂量',
                    type: 'line',
                    smooth: true,
                    itemStyle: { normal: { areaStyle: { type: 'default' } } },
                    data: [100, 200, 150, 130, 260, 830, 710]
                },
                {
                    name: '产奶量',
                    type: 'line',
                    smooth: true,
                    itemStyle: { normal: { areaStyle: { type: 'default' } } },
                    data: [30, 182, 216, 156, 390, 300, 356]
                }
            ]
        },
        media: [
            {
                query: {},
                option: {
                    // 桌面端配置
                }
            },
            {
                query: { maxWidth: 400, ismobile: true },
                option: {
                    // 移动端配置
                    legend: {
                        right: 'center',
                        bottom: 0,
                        orient: 'horizontal'
                    },
                    toolbox: {
                        orient: 'vertical'
                    }
                }
            }
        ]
    };

    myChart.setOption(option);
}
```

以上就是对 ECharts 在移动端的优化与自适应方案的介绍。通过合理运用 ECharts 提供的定位、布局和 Media Query 功能，再结合 JavaScript 对浏览器信息的识别，我们可以实现图表在不同尺寸屏幕上的最佳展示效果。
