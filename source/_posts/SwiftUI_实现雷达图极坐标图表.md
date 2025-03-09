---
title: SwiftUI 实现雷达图/极坐标图表
cover_image: /img/cover.png
abbrlink: 2617abaa
date: 2023-10-29 15:56:20
---

---
## SwiftUI-Radar-Chart

SwiftUI 实现雷达图/极坐标图表

SwiftUI 团队十分贴心地在 WWDC 2022 上发布了一个叫做 SwiftUI Charts 的框架，其中包含了一些常见的图表，比如折线图、柱状图、饼图等等。但是雷达图/极坐标图表并没有在其中，最近正好有个小项目需要用到极坐标图，于是就自己实现了一个。

## 实现

虽然 SwiftUI Charts 没有直接提供雷达图/极坐标图表，但是 SwiftUI 中还是可以比较方便的绘制路径，通过 Path 和 Shape 协议就可以实现。

### 极坐标图

首先定义一个极坐标点

```
struct PolarPoint         while tmp < 0         self.radian = tmp    }}
```
其中初始化时把角度都调整到了 ，方便后续排序展示

#### 坐标系

```
struct PolarChartGrid: Shape             path.move(to: center)        for step in 1 ... divisions             return path    }}
```
Path.addArc 函数可以很方便的添加圆弧

#### 数据

由于数据这里都是极坐标格式表示的，但是绘制是基于直角坐标，展示时要进行转换，并且 UI 系统中的 y 轴与数学上惯用的 y 轴方向是相反的，转换时要注意

```
struct PolarChartPath: Shape         }        return path    }}
```
#### 合成

最后将坐标系与数据叠加在一起就完成了。

这里在总 View 的 data 属性初始化时对数据进行了排序，这样可以保证绘制时的顺序，并且在总 View 处进行排序可以避免以后对 Shape 做动画时数据的重复排序节省计算资源。

```
struct PolarChart: View )        self.gridColor = gridColor        self.dataColor = dataColor    }      var body: some View )!.radius, data: data)                .stroke(lineWidth: 1.0)        }    }}
```
### 雷达图

完成极坐标图后再完成雷达图就简单了，只需坐标系绘制时将圆弧改为分段直线，另外数据的格式也可以简化为直接使用 Double 数组表示

#### 坐标系

```
struct RadarChartGrid: Shape             for step in 1 ... divisions         }            return path    }}
```
#### 数据

```
struct RadarChartPath: Shape             let radius = min(rect.maxX - rect.midX, rect.maxY - rect.midY)        var path = Path()            for (index, entry) in data.enumerated()         }        path.closeSubpath()        return path    }}
```
#### 合成

```
struct RadarChart: View       var body: some View     }}
```
## 小结

本文实现了一个简单的雷达图/极坐标图表，但是还有很多可以改进的地方，功能上可以增加动画、增加数据标签、增加数据点标签，工程实现上可以利用 Swift 作为 DSL 的优势，利用 ViewModifier、ViewBuilder 等特性，将数据与样式分离，实现更好的复用性。

