---
title: 关于Figma组件属性的一点点
cover_image: /img/cover.png
abbrlink: 633479c1
date: 2022-12-16 02:26:13
---

关于Figma组件，最基础的使用是创建一个父级组件，再复制出子组件放到各页面中，其中子组件可以修改文字和颜色。但是随着文件内容增多，我们希望子组件能更加灵活，以适用更多的场景。Figma在今年上半年更新的组件属性功能就可以达到这个效果。下面根据我的一些使用经验，结合组件属性的基本内容，介绍能方便地替换子组件中icon、素材图片、文字内容的方法。## 关于组件的图层

首先介绍一下围绕组件，会存在的几种图层样式：**父级组件（main component）：**单个的父级组件只有一个样式，图层icon是四个小方块。**组件变体（variant）：**多个父级组件也可以组成一个“父级组件组”的概念，这个“父级组件组”的图层icon是四个小方块，其中的每个组件叫做组件变体。**子级组件（component）：**由父级组件复制出来的，可以放在各页面中使用。![](https://mmbiz.qpic.cn/mmbiz_png/M5SYxpNaTK3Whmicw1z9JQ3iciavF3wEpCj7vjCGgwsWKVibEN60CQPCG0XyJukRTBHmTtKHXMYPIaLtI9hJ9qB7GQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)## 

## 基础修改内容

颜色、文字、有时Frame可以更改fixed/hug/fill

## 换图片的方法

在平面设计的软件里，我们经常使用蒙版去将图片切割成想要的形状。但用这种方法在Figma组件中添加/更换图片时会比较麻烦。正确的添加图像方法：选中矢量图形，在颜色填充（Fill）中，将 Solid 改成 Image，再选择图片文件就可以了。并且图片支持裁剪、放大缩小等操作方便调试。![](https://mmbiz.qpic.cn/mmbiz_png/M5SYxpNaTK3Whmicw1z9JQ3iciavF3wEpCjibPUzHpuic5pJuiauHrGGDRMktKjMtBibTqHrVkF6LhLLzVRzgsExSCswQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)## 变体及其使用

想要更自由地使用组件的功能，需要先了解组件变体及其四个属性。

## 添加变体（variant）

方法1: 创建一个 main component 后，点击Figma顶部的“添加变体（Add variant）”或者右击找到“Main component —— Add variant”。

方法2: 如果已经存在变体，鼠标点击到其中一个variant时，点击下方出现的紫色小加号➕即可。复制出来的新 variant，会保留内部的 Instance swap 和 Text属性的值。

方法3: 先绘制好几种状态，全选后在 Figma 顶部点“Create Component Set”。

**需要注意的是：**点击父组件，在右侧属性（Properties）中，点击加号➕，也可以找到Variant，但这里的意思是添加了一层变体属性（Variant），和上面3种添加变体产生的效果不同，具体会在下面讲到。

## 组件属性（Properties）

官方的名称理解起来有点抽象，所以我在属性名称后面破折号写了一些主要用法解释。

### 1.  变体属性（Variant）——给组件分类整理

点击父组件，在右侧属性（Properties）中，点击加号➕，可以找到 Variant看不到Properties的话，先确认一下鼠标是否点在了main component上（而不是在其中的 variant上），注意看左侧图层前的icon是否为四个紫色小方块。

它的作用主要是给组件分类整理，这样在组件数量多时，子组件可以快速筛选到想要的变体效果。举例：如图是在TDesign for web (Community)里的截图，它将导航菜单项根据：是否有图标、显示状态、是否为禁用、是否有分组等进行分类。在子组件使用时，就可以根据需要选择合适的样式。

（图源https://www.figma.com/community/file/1170935033971188639）

如上图Current variant中，左侧一列可以理解为分类标准（Name），右侧一列为类别下的属性值（Value）。属性值有两种显示方式：一种是输入后成为选项，在子组件属性中下拉选择，适合属性值数量大于3的情况。另一种可以用键盘输入“on/off”，属性值会显示为开关，更适合某分类中只有2个值的情况。

### 

### 2.  布尔属性（Boolean）——属性面板控制图层可见性

添加了布尔属性的图层，可以控制该图层的可见性。效果相当于可以在子组件的属性列表直接控制该图层是否可见（Show/Hide）（对图层使用Shift+Ctrl+H的效果）。

### 

### 3.  实例交换属性（Instance swap）——设置局部可替换组件——换icon的方法

子组件中 icon 不可以直接替换，因为受父组件的限制。

替换的方法：回到 main component 中，将需要替换的部分添加 Instance swap 属性。即可在子组件使用时，保留该部分图层布局，选择替换为其他组件（⚠️被替换的内容提前设为 main component，才能被浏览到）。

### 

### 4.  文本属性（Text）——属性面板直接改文字

比较简单，给文字图层添加 Text 属性，可以在属性面板修改文字，不用一个个点进图层修改。还有一种用法是可以批量修改文字，但是个人使用不多，具体方法可以去官方教程查看。（官方教程：https://www.figma.com/community/file/1100581138025393004）

### 制作步骤

因为复制出来的变体会带有各个属性，所以可以提前预估最后的效果，在前面的 variant 中就添加好属性，这样减少先复制再逐个添加属性的麻烦。

## 组件嵌套组件

有时会遇到在 main component 中看不到添加属性的按钮，或者属性按钮是禁用状态的情况。这可能是因为该父组件中还嵌套了别的子组件。修改的时候记得回到该子组件对应的父组件中就可以啦。

举例：现在评论管理的 Text 属性是禁用状态，因为“第二层组件”中还嵌套了“导航菜单”的子组件，所以应当回到“导航菜单的 main component”去修改属性。

## 总结

写这篇文章用的一个小例子，效果如下：

