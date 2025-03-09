---
title: CSS 实践方式
cover_image: /img/cover.png
abbrlink: 7b4da572
date: 2022-04-01 15:37:57
---

Tailwind CSS，一个功能类优先的 CSS 框架。

基于 Tailwind CSS 的核心概念，了解 CSS 中的一些设计理念。

在面向对象的程序设计中，我们普遍遵循高内聚、低耦合的原则。一个程序或项目往往由多个模块组成，高内聚指一个模块中各个元素之间联系较为紧密，从而意味着更高效的功能实现；低耦合指每个模块之间联系的紧密程度较低，意味着单个模块独立性更强，可重用性与可移植性更高，便于后续的维护。

简单来说，HTML 负责结构和内容，CSS 负责样式。理想情况下，它们之间应该分工明确，互不干扰，显然这是不切实际的。

随着技术的发展，为了使项目结构更加清晰，逻辑更加合理，可维护性更高，产生了各种各样的实践方式。

### HTML 与 CSS 间的实践方式

#### 不提倡的方式

##### 内联样式

内联样式，将样式写入 HTML 标签中，一方面来说其较高的优先级使得样式间的继承关系变得更为复杂，另一方面来说在耦合性较高的同时，与 内部样式 或 外联样式 相比，并没有鲜明的优势，维护起来更加困难，且无法使用伪类或伪元素，灵活性较差。

##### CSS 结构化命名

结构化命名，根据页面中模块的位置命名。

```
<div class="content-left">  <div class="text-center">div>div><div class="sidebar">div>
```
该例中，当希望将 sidebar 移到左侧时，需要同时修改选择器的名字，否则就会与实际情况产生矛盾，特别是在页面结构相对灵活和复杂时，情况更加难以琢磨。

#### 关注点分离 与 CSS 语义化命名

语义化命名，根据模块的功能命名，从而 HTML 只包含关于内容的信息，所有关于样式的决定都在 CSS 中进行，这意味着只需要更换样式表，就可以重新设计一个网站。

```
<div class="main">div><div class="sidebar">div>
```
这是一个相对宽泛的概念，不加以约束时，存在以下几个问题：

* 命名冲突。

* 使用子代选择器时，HTML 与 CSS 间仍然有很强的耦合性，嵌套的 CSS 映射了 HTML 的结构。

HTML 并不关心样式决定，但 CSS 却非常关心 HTML 结构。

* 适合的名字很难起，特别是英语水平欠佳的时候（或许可以有效提升英语水平）。

#### BEM

Block Elenment Modifier，简称 BEM，是一种 CSS 命名规范。

BEM的命名规则：block-name__element-name--modifier-name，模块名 + 元素名 + 修饰器名。

```
<form class="search-form">        <input class="search-form__input">         <button class="search-form__button">搜索按钮button>form>
```
```
<div class="banner__btn">  <button class=".button .banner__btn--red">button>    <button class=".button .banner__btn--green">button>div>
```
```
.banner__btn--red .banner__btn--green 
```
##### 解决的问题一 命名冲突

CSS 是全局化生效的，单独开发组件时，如果不在编译时进行处理，如使用 Webpack 的 css-loader，容易产生命名冲突，从而导致样式冲突。

理想情况下，在一套组件的开发过程中，我们只需考虑组件内的命名情况，而不必担心是否和组件外产生冲突。

解决思路，在同一个项目中，每个组件都有其独特的命名，从而在为组件内部元素命名时加上组件名，组件内的样式就不会与组件外的样式冲突。

##### 解决的问题二 子代选择器 CSS 对 HTML 结构上的依赖

使用子代选择器时，用层次关系结构来定位元素，可能会因为需求改变而大面积重写。同一个元素样式零散分布在文件的不同地方，定位该元素的选择器也可能各不相同。

后期维护时，需要一边检查该元素的 DOM 结构，一边对照 CSS 文件，找到对应样式，再进行更改。

权重问题，在响应式布局中，通过覆盖之前的元素样式来达成效果，此时需要不断确认该元素之前的选择器写法来计算它的权重，同时以相同或更高的权重来覆盖，甚至需要通过添加额外的类名或标签名。

解决思路，BEM 禁止使用子代选择器，在命名过程中，它是不考虑结构的，父元素名、模块构造或是元素之间层级关系互相变动，都不会影响元素的名字，一个元素总是一个模块的一部分，而不是另一个元素的一部分。此时，组件内所有元素的权值都是统一清晰的，便于响应式中覆盖样式。

##### 开放封闭原则

对扩展是开放的，对修改是封闭的。在 BEM 中，我们可以理解为，应该使用 modifier 去拓展样式，而不应该去修改 block 或 element 的基础样式。

```
<div>    <button class="btn btn--green">正确button>    <button class="btn btn--red">错误button>div>
```
```
.btn .btn--green .btn--red 
```
##### SEM 与 BIO

Scalable Extensible Maintainable，可伸缩的 可扩展的 可维护的

BEM ITCSS OOCSS

此时，CSS 与 HTML 结构不再耦合。

#### 新的问题 处理类似的组件

这里有一张介绍宠物类型的卡片：

```
<div class="pet-introduction">  <img class="pet-introduction__image" src="https://tse1-mm.cn.bing.net/th/id/R-C.ba505e59ca3243fb4027fdf29eae9cea?rik=oQTn%2fbheyUTQOA&riu=http%3a%2f%2fpicture.ik123.com%2fuploads%2fallimg%2f191129%2f12-191129143024.jpg&ehk=YrFqIM1jz%2fzt9a4M7CuO5TKex1YbAMZ9Zujl5sDV5PE%3d&risl=&pid=ImgRaw" alt="">  <div class="pet-introduction__content">    <h2 class="pet-introduction__name">Rabbith2>    <p class="pet-introduction__body">      Rabbits are small mammals in the family Leporidae (along with the hare) of the order Lagomorpha (along with the pika). Oryctolagus cuniculus includes the European rabbit species and its descendants, the world's 305 breeds[1] of domestic rabbit.    p>  div>div>
```
```
.pet-introduction .pet-introduction__image .pet-introduction__content .pet-introduction__name .pet-introduction__body 
```
现在我们有一个新的需求，需要将文章预览以卡片的形式展现。

我们不能将 .pet-introduction 直接应用到新的卡片中，这样不符合语义，.article-preview 应该是独立的组件。

##### 处理方案一 复制样式

```
<div class="article-preview">  <img class="article-preview__image" src="https://upload.wikimedia.org/wikipedia/commons/a/ad/BolexH16.jpg" alt="">  <div class="article-preview__content">    <h2 class="article-preview__name">The History of Filmh2>    <p class="article-preview__body">      A film, also called a movie, motion picture or moving picture, is a work of visual art used to simulate experiences that communicate ideas, stories, perceptions, feelings, beauty, or atmosphere through the use of moving images.    p>  div>div>
```
```
.article-preview .article-preview__image .article-preview__content .article-preview__name .article-preview__body 
```
##### 处理方案二 @extend

使用预处理器的 @extend 功能，进行复用，一般不建议这样使用

##### 处理方案三 抽出一个与内容无关的组件

从设计角度看，这两个组件有很多共同点，但从语义角度看，这两个组件的功能各不相同。

从而我们可以抽出一个覆盖它们共同点的新组件，称为 .media-card。

```
.media-card .media-card__image .media-card__content .media-card__name .media-card__body 
```
这样我们的 HTML 就可以改写为：

```
<div class="media-card">  <img class="media-card__image" src="https://tse1-mm.cn.bing.net/th/id/R-C.ba505e59ca3243fb4027fdf29eae9cea?rik=oQTn%2fbheyUTQOA&riu=http%3a%2f%2fpicture.ik123.com%2fuploads%2fallimg%2f191129%2f12-191129143024.jpg&ehk=YrFqIM1jz%2fzt9a4M7CuO5TKex1YbAMZ9Zujl5sDV5PE%3d&risl=&pid=ImgRaw" alt="">  <div class="media-card__content">    <h2 class="media-card__name">Rabbith2>    <p class="media-card__body">      Rabbits are small mammals in the family Leporidae (along with the hare) of the order Lagomorpha (along with the pika). Oryctolagus cuniculus includes the European rabbit species and its descendants, the world's 305 breeds[1] of domestic rabbit.    p>  div>div>
```
这种方式消除了 CSS 中的重复，但是又将 HTML 与样式关联了起来，此时如果想单独给其中的一种卡片定制样式，就需要到 HTML 中再进行修改，修改方式可以考虑上文提到的开放封闭原则。

#### 并不存在真正的 关注点分离

由上文可得，并不存在真正的关注点分离，只是依赖的方向不同，我们有两种方式来处理 CSS 与 HTML 的关系。

##### 依赖于 CSS 的 HTML

HTML 是独立的，它只负责提供需要 CSS 操作的功能块，CSS 针对这些功能块设计样式。

意味着可重塑的 HTML。

##### 依赖于 HTML 的 CSS

使类的命名与内容无关，而是与重复样式有关，CSS是独立的，它只负责提供 HTML 需要的样式，由 HTML 去进行组合，实现所需要的设计。

意味着可重用的 CSS。

根据自己的需要作出选择：可重塑的 HTML，还是可重用的 CSS？

> CSS功能类与关注点分离 | TailwindCSS 中文网 (tailwindchina.com)
> 
> #### 功能类优先的 CSS

这里以 Tailwind CSS 为例。

将可重用的 CSS 细粒化，比起重复，我们更喜欢组合。

```
<div class="p-6 max-w-sm mx-auto bg-white rounded-xl shadow-md flex items-center space-x-4">  <div class="flex-shrink-0">    <img class="h-12 w-12" src="/img/logo.svg" alt="ChitChat Logo">  div>  <div>    <div class="text-xl font-medium text-black">ChitChatdiv>    <p class="text-gray-500">You have a new message!p>  div>div>
```
功能类优先的 CSS 乍一看与 内联样式 十分相似，然而它有着 内联样式 遥不可及的优势。

##### 强制一致性

使用小型、可组合的功能类时，同一团队的开发人员总是从一组固定的选项中进行选择，免于出现同样是灰色的文本，其实是五彩斑斓的情况（假如没有设计），有效限制了 CSS 的规模随着项目的大小而线性增长。

一个新的 CSS 块是一张空白的画布，自由而危险。

##### 仍然可以提取组件

提升可维护性，解决 CSS 中的重复性问题。

##### 响应式设计与状态处理

或许可以看作是一种 CSS in HTML 的模式，维护功能优先的 CSS 项目比维护大型 CSS 代码库更加容易。

