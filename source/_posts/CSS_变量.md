---
title: CSS 变量
cover_image: /img/cover.png
abbrlink: 6bf17bf4
date: 2022-05-13 14:04:14
---

自定义属性（CSS 变量或者层叠 variables）是由 CSS 作者定义的实体，这些实体在一个 document 内可以被重用。其目的是可重用性并减少 CSS 值中的冗余。

一般按照自定义属性的符号设置 （比如，--main-color: black；） 然后使用 var()函数使用 （例如 color: var(--main-color)）

* CSS 变量可以访问 DOM，这意味着可以创建具有局部或全局范围的变量，使用 JavaScript 来修改变量，以及基于媒体查询来修改变量。
### CSS变量的好处

1. 可维护性 在一个Web开发项目中，我们经常重复使用一个特定的CSS属性值：
```
h1 p img .callout 
```
复杂的网站有大量的 CSS，通常会有很多重复的值。例如，同一个颜色可能会被在几百个地方都用到，可以

当需要改变某些属性值时，问题就会暴露出来了。如果我们手动地改变属性值，尤其是当CSS文件很大时，不仅会花费大量时间，还可能会犯一些错误。

同样的，如果我们从全局搜索去一次性替换掉理论上可行，但如果只需改变特定地点的颜色,全局搜索就可能会无意中影响其他样式规则。

我们可以使用CSS变量重写：

```
:root h1 p img .callout 
```
假设客户需求说由于文本颜色太亮，导致文本难以阅读，想要改变文本颜色，此时，我们只需要更新一行CSS规则：

```
--text-color: black;
```
1. 提高CSS的可读性 自定义属性会使样式表更加易读，也会使CSS属性声明更有语义。将这个声明
```
background-color: yellow;font-size: 18px;
```
和下面的声明比较

```
background-color: var( --highlight-color );font-size: var( --base-font-size );
```
像yellow和18px一类的属性值并没有给我们任何有用的上下文信息. 但是当我们阅读如--base-font-size和--highlight-color一样的属性名时，即便在其他人的代码，我们都能马上知道这个属性值是在做什么,尤其是这个值在其他的上下文中也存在时。。

### 命名变量

与编程语言命名变量相似，CSS 变量的有效命名应包含字母数字字符，下划线和破折号。另外，CSS 变量区分大小写。

```
:root .box .circle 
```
```
/* 合法命名 */:root /* 非法命名 */:root 
```
### 作用域

CSS 变量也有自己的作用域，这个概念类似于其他编程语言。

```
:root .section-title 
```
--primary-color 全局变量可以从文档中的任何地方访问。若变量--primary-color 在.section-title 定义，在作用域.section-title 中新定义的会覆盖全局的定义。

### eg1：控制组件的大小

在设计系统中，按钮通常有多种尺寸。通常，按钮可以具有三种尺寸（Small, normal, large）。使用 CSS 变量来实现：

```
.button .button--small .button--large 
```
通过在按钮组件作用域内更改变量--unit，我们创建了按钮的不同变体。

### eg2：CSS 变量和 HSB 颜色

HSB(Hue-saturation-brightness) 代表色调，饱和度，亮度。色相的值决定了颜色，饱和度和亮度值可以控制颜色的深浅。

```
:root .button /* 使背景更暗 */.button:hover 
```
可通过减小变量--primary-b 使按钮变暗。

### eg3：动态应用于CSS Grid

CSS 变量对于网格非常有用。假设希望网格容器根据定义的首选宽度显示其子项。与为每个变体创建类并复制CSS相比，使用变量更容易做到这一点。

```
.wrapper .wrapper-2 
```
这样，我们可以创建一个完整的网格系统，该系统灵活，易于维护，并且可以在其他项目中使用。可以将相同的概念应用于grid-gap属性。

```
.wwrapper .wwrapper.gap-1 
```
事实上，若是不介意用一点行内样式表，可以省去一个类的添加显得更加精简

```
<div class="wrapper" style="--item-width: 250px;">  <div>div>  <div>div>  <div>div>div>
```
```
.wrapper 
```
### eg4:全值声明，CSS 渐变

以全值表示，例如，类似渐变的东西。如果整个系统中使用渐变或背景，可将其存储到CSS变量中。

```
:root .element 
```
或者我们可以存储一个值。以角度为例:

```
.element .element.inverted 
```
### eg5: 在明暗模式之间切换

纯CSS实现，不必借助js的读取与修改,不必专门复制粘贴写多个类进行类名的转换 (夜间模式的实现)

```
:root /* 添加到``元素的类*/.dark-mode 
```
### eg6:图片大小调整与CSS里的计算

假设我们需要四种不同大小的图片，并且只能使用一个变量来控制其大小。

```
<img src="user.jpg" alt="" class="c-avatar" style="--size: 1" /><img src="user.jpg" alt="" class="c-avatar" style="--size: 2" /><img src="user.jpg" alt="" class="c-avatar" style="--size: 3" /><img src="user.jpg" alt="" class="c-avatar" style="--size: 4" />
```
```
.c-avatar 
```
计算值 要查看CSS变量的计算值，只要将鼠标悬停或单击即可。

### eg7:媒体查询

根据视口大小更改其间距 如下例当浏览器的宽度为 450px 或更宽时，将 --fontsize 变量值设置为 50px，将 --blue 变量值设置为 lightblue。

```
.container @media screen and (min-width: 450px)   :root }
```
### eg8:继承

如果父元素中定义了CSS变量，那么子元素将继承相同的CSS变量。.child元素可以访问变量--size，因为它从父元素继承了它。

```
<div class="parent">  <p class="child">p>div>
```
```
.parent .child 
```
假设有一组以下需求的操作项

* 改变一个变量就可以改变所有项的大小
* 间距是动态的
```
<div class="actions">  <div class="actions__item">div>  <div class="actions__item">div>  <div class="actions__item">div>div>
```
```
.actions .actions--m .actions__item 
```
子代盒子类action__item继承父亲的--size变量 也可以应用在动画里 不需要定义@keyframes多次，它将继承.big和.small元素的定制CSS 变量

```
@keyframes breath   to }.big .small 
```
```
<div class="big" >biggerdiv><div class="small" >smallerdiv>
```
### 回退方案

这里的回退不是不支持 CSS 变量的回退，而是 CSS 变量可以支持回退方案。

```
.section-title 
```
注意，var()有多个值。第二个#222 只在变量--primary-color 由于某种原因没有定义的情况下有效。不仅如此，我们还可以将 var()嵌套到另一个 var()中。

```
.section-title 
```
在变量值依赖于某个动作的情况下，该特性非常有用。当变量没有值时，为它提供一个回退很重要。

##### 无效值处理

谨防给CSS属性分配错误类型的值。在下面的示例中，由于自定义属性--small-button被赋予一个长度单位，它被用在.small-button样式中作为color的属性值是无效的

```
:root .small-button 
```
避免这种情况的一个好方式是想出具有描述性的自定义属性名称。例如将上面的自定义属性命名为--small-button-width

当var()函数中的CSS变量无效时，浏览器将根据所使用的属性用初始值或继承值替换。由于color属性是继承的，因此浏览器将执行以下操作：

* 该属性是否可继承？

* 是的，继承该值
* 否:设置为初始值
* 如果是，父节点是否拥有该属性?
* 否:设置为初始值
### 网址值

考虑将链接的URL值存储在CSS变量中。语义化提高可读性特征明显，哪个位置引入哪个链接。便于引入外部资源。

```
:root .section 
```
### 存储多个值

```
:root .section-title 
```
在示例中，我们有一个rgba()函数，并且RGB值存储在CSS变量中，以逗号分隔。如果我们想根据元素调整alpha值，这样做可以提供灵活性。

### JavaScript 操作

css与js的联动

```
// 设置变量document.body.style.setProperty('--primary', '#000');// 读取变量document.body.style.getPropertyValue('--primary');// '#000'// 删除变量document.body.style.removeProperty('--primary');// 从行内样式中中获取变量element.style.getPropertyValue("--my-var");// 从任何地方获取到变量getComputedStyle(element).getPropertyValue("--my-var");
```
### 兼容性和查看浏览器的支持问题

可以通过css语法来检测浏览器是否支持css变量 通过@supports性能查询语法来检测：

```
@supports (--a: 0)}@supports(not(--a: 0))}
```

