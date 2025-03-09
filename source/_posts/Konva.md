---
title: Konva
cover_image: /img/cover.png
abbrlink: 8bdd1084
date: 2022-06-10 15:37:58
---

Konva 是一个 HTML5 Canvas JavaScript 框架，它通过对 2d context 的扩展实现了在桌面端和移动端的可交互性。## 基本层次

渲染分层清晰：Stage，Layer，Group 和 Shape

我们可以用画布类比 Stage，图层类比 Layer，图层中的元素就是 Shape，Group 则是几个元素的组合。

官方给出的层次体系如下所示：

```
              Stage                |         +------+------+         |             |       Layer         Layer         |             |   +-----+-----+     Shape   |           | Group       Group   |           |   +       +---+---+   |       |       |Shape   Group    Shape           |           +           |         Shape
```
## 绘制图形与设置样式

Konva 支持的图形：Rect，Circle，Ellipse，Line，Polygon，Spline，Blob，Image，Text，TextPath，Star，Label，SVG Path 和 RegularPolygon

样式：

```
let tr = new Konva.Transformer();tr.attachTo(e.target);
```
### 自定义图形 custom shape:

Konva.Context 是对原生 2d canvas context 的包装，它除了和原生 context 有相同的属性和方法外还增加了一些额外的 API。

```
var rect = new Konva.Shape(});
```
最佳实践：尽可能不要在 sceneFunc 里面手动添加样式，你只需要绘制一些路径，然后使用 context.fillStrokeShape(shape) 处理样式问题。

## 绑定事件、交互与变换

用 on() 给节点绑定事件，基本支持鼠标键盘以及移动端事件。

事件也包含：dragstart、 dragmove、 dragend 等拖拽事件，transformstart, transform 和 transformend 变换事件。

## 最简单的一个例子：

```
let stage= new Konva.Stage();  let layer= new Konva.Layer();let group= new Konva.Group();let rect= new Konva.Rect();group.add(rect);layer.add(group);stage.add(layer);
```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rz25qKmt049zvsKjNd6KibITOUbAGm82qXAswQsLicD2dsUMSHISXYyT4TySP3BcTgZ7QZPwMef3dQ/640?wx_fmt=png)### 添加事件

```
rect.on("mouseenter mouseup", function () );rect.on("mouseleave  mousedown ", function () );
```
### 添加键盘事件

```
let container=stage.container();container.tabIndex=1;//使其可以被focusconst DELTA=4;container.addEventListener("keydown",function(e)else if(e.keyCode===38)else if(e.keyCode===39)else if(e.keyCode===40)else    e.preventDefault();    layer.batchDraw();});
```

