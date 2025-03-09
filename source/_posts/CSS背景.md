---
title: CSS背景
cover_image: /img/cover.png
abbrlink: 28d95575
date: 2023-11-19 13:50:37
---

## 推荐一本好书

书名：《CSS揭秘》

![](https://mmbiz.qpic.cn/mmbiz_jpg/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjy3Tx31nbYrjDodKxB4juJZtNF7srNG6MNzG1ia5fPPJwhqVlKicwzWTibw/640?wx_fmt=jpeg&from=appmsg)作者简介：

Lea Verou

> W3C CSSI作组特邀专家，设计CSS语言的委员之一，此前曾在W3C担任开发者代言人。目前，她在麻省理工学院从事人机交互领域的研究。她还是一位博客作家，并经常在国际性技术会议上担任讲师；她创建的多个开源项目广受开发者欢迎。
> 
> 定位

适合CSS进阶，或者对CSS使用技巧有浓厚兴趣的人。这本书不是一本“菜谱”，而是以某个专题为切入点，深入讲解一些实现层面的细节，而且极为优雅，或者说有“工匠精神”

## from WET to DRY

这里 wet 和 dry 显然有更深层的含义， wet 展开是：”We Enjoy Typing“ ， DRY 展开是 : “Don’t Repeat Yourself”。

这两个词在所有编程语言的教程中几乎都能够看到，CSS虽然不算一门编程语言，但是其实很多时候编写CSS的思维模式和编写代码有共通之处，写出简洁、易维护、易修改的CSS是我们的美好期望。

### 合理使用简写属性

下面的代码：

```
background:rebeccapurple;background-color:rebeccapurple;
```
这两个写法其实是有区别的。

前者能够确保我们得到一个紫色的纯色背景图，而后者却未必，因为它只是声明了颜色，如果同时有一个background-image的声明在起作用，那么它得到的就可能是一只小猫的图片。

简写的意义在于：

1. 对于我们没有在简写中显式给出的属性，会被给予默认值。（如background-image 会被设置为 none)
2. 如果未来css引入新的特性，那么简写能够自动涵盖新的属性，可以抵御未来的风险。
3. 合理简写能够减少重复
这里我一直在强调合理

因为一味追求简写也会导致代码变WET，比如下面的例子：

```
background:url(a.png) no-repeat top right /2em 2em,            url(b.png) no-repeat bottom right /2em 2em,            url(c.png) no-repeat bottom left /2em 2em;
```
background-size和background-repeat的值被重复了三遍，尽管每层背景的这两个值确实是相同的

这里我们就可以用到“列表扩散规则”（如果只为某个属性提供一个值，那它就会扩散并应用到列表中的每一项）

```
background:url(tr.png) top right,            url(br.png) bottom right,            url(bl.png) bottom left;background-size:2em 2em;background-repeat:no-repeat;
```
这样的话如果我们日后要修改，就只需要修改两处，而不是原来的六处了。

### 被滥用的浏览器前缀

浏览器前缀是对CSS实验性特性的解决方案，但它饱受诟病，这个设计的最初意图是：

> 每个浏览器都可以实现这些实验性的（甚至是私有的、非标准的）特性，但要在名称前面加上自己特有的前缀。最常见的前缀分别是Firefox的-moz-、IE的-ms-、Opera的-o-以及Safari和Chrome的-webkit-。网页开发者可以自由地尝试这些加了前缀的特性，并把试用结果反馈给工作组，而工作组随后会将这些反馈吸收到规范之中，并且逐渐完善该项特性的设计。由于最终标准化的版本会有一个不同的名称（没有前缀），它在实际应用中就不会跟加前缀版本相冲突了。
> 
> 由于不同浏览器支持这些特性的速度是不一样的，意味着如果有一个厂商支持了某个实验性特性，开发者就要在代码中追加对应前缀的样式，于是开发者想了一个办法，就是一次性把所有前缀都加上，最后加个不带前缀的版本，最后代码看起来像这样：

```
-moz-border-radius:10px;-ms-border-radius:10px;-o-border-radius:10px;-webkit-border-radius:10px;border-radius:10px;
```
> 这里面有两条声明是完全多余的：-ms-border-radius和-o-border-radius这两个属性从来没有在任何浏览器中出现过，因为IE和Opera从一开始就是直接实现border-radius这个无前缀版本的
> 
> 显然，把每个声明都重复五遍是相当枯燥的，而且很难维护。更要命的是，如果需求发生改动，你就需要改动这么多处的代码。

代码变得非常WET !

## 从0开始的渐变图像冒险

突击提问：你用过 conic-gradient 吗？

### linear-gradient 线性渐变

简单例子

线性渐变的渐变方向是一条直线，设置好色标，角度，就能得到一个不错的效果：

```
background: linear-gradient(red 0, blue 100%)/* 默认从0开始，到100结束， 下面的等价 */background: linear-gradient(red, blue)
```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjytGDKA2MtBEdewpbNMHEZNpoicHkl9sxIksHDm8h3H6wWBF1jVddb67g/640?wx_fmt=png&from=appmsg)💡 渐变色应该用于background-image 而非 background-color

条纹

合理设置色标可以避免渐变区域的产生，从而产生条纹。

```
background:linear-gradient(#fb3 50%,#58a 50%);background-size: 100% 30px;
```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjyib1UdIhvMeEScrAwhsribp4gwIqLpVTprMWdYJW1lScDt1VqcbXWdDSw/640?wx_fmt=png&from=appmsg)或者是用repeating-linear-gradient

```
/* 几种等价的写法 */background: repeating-linear-gradient(#fb3 0, #fb3 15px, #58a 15px, #58a 30px);background: repeating-linear-gradient(#fb3, #fb3 15px, #58a 15px, #58a 30px);background: repeating-linear-gradient(#fb3 0, #fb3 15px, #58a 0, #58a 30px);
```
效果和前面完全一样，repeating前缀的版本会自动重复，无需设置size

💡 仔细看第三个写法，这是正确的吗？绝对正确，这是因为CSS规范规定：“如果某个色标的位置值比整个列表中在它之前的色标的位置值都要小，则该色标的位置值会被设置为它前面所有色标位置值的最大值”，当我们设置为0时，会自动使用15px 作为色标，这样写的好处在于，如果我们要改变条纹的宽度，只需要修改一处。

改变方向

```
/* 下面两个写法等价 */background: repeating-linear-gradient(to right, #fb3 0, #fb3 15px, #58a 0, #58a 30px);background: repeating-linear-gradient(90 deg, #fb3 0, #fb3 15px, #58a 0, #58a 30px);
```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjybJYvn4icudcKSQ068ictPp0ph010bhJ6jmkU0GgwHrapWvQ88C6tKDiaA/640?wx_fmt=png&from=appmsg)### 更加复杂的图像

单个背景图能做到的效果是有限的，但是如果我们把多个渐变图叠加在一起，并且充分利用遮罩关系，或者改变backgroun-blend-mode就能产生许多复杂的背景图

> CSS3 Patterns Gallery (verou.me)
> 
> 比如下面的代码：

```
linear-gradient(45deg, #92baac 45px, transparent 45px)64px 64px,linear-gradient(45deg, #92baac 45px, transparent 45px,transparent 91px, #e1ebbd 91px, #e1ebbd 135px, transparent 135px),linear-gradient(-45deg, #92baac 23px, transparent 23px, transparent 68px,#92baac 68px,#92baac 113px,transparent 113px,transparent 158px,#92baac 158px);background-color:#e1ebbd;background-size: 128px 128px;
```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjyaSrghtFkLI9xF3w0QfmoLtdfRickfGwKDviavEKuRX0shUwr95lo3MOQ/640?wx_fmt=png&from=appmsg)### 画一个棋盘

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjyCicp3IQ9N8zxIVicHOfTvNkJLaPDSaHIIXAgyb7KcmQZ4WWrbfCiaVtWA/640?wx_fmt=png&from=appmsg)有思路吗？

只观察这个图片的基本组成单位，我们容易发现这是一个2 * 2的重复结构

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjy12s2O5nQqkRfMF7R4ghxgiaUvgAGwI6SKWB5NHVvibFF1dnurwaye0ow/640?wx_fmt=png&from=appmsg)可以用三角形拼接，再加上错位，可以合成两个灰色正方形，把白色作为底色，就能得到棋盘效果：

```
background: #eee;      background-image:        linear-gradient(45deg,          rgba(0, 0, 0, .25) 25%, transparent 0,          transparent 75%, rgba(0, 0, 0, .25)0),        linear-gradient(45deg,          rgba(0, 0, 0, .25) 25%, transparent 0,          transparent 75%, rgba(0, 0, 0, .25)0);      background-position: 0 0, 15px 15px;      background-size: 30px 30px;
```
有没有更好的办法？

有！

### conic-gradient

这个又叫锥形渐变，画过色轮的人应该不陌生。它的渐变色产生的过程是：渐变的颜色围绕一个中心点旋转（而不是从中心辐射）进行过渡。

中心位置是可以调的，这就使得这个函数异常强大。

```
background: conic-gradient(#bbb 0, #bbb 25%, #eee 0, #eee 50%, #bbb 0, #bbb 75%, #eee 0, #eee);background-size:30px 30px;
```
效果和之前完全一样，但是代码变的很简洁

还可以进一步优化：

```
background:repeating-conic-gradient(#bbb 0,#bbb 25%,#eee 0,#eee 50%);background-size:30px 30px;
```
改变中心位置

```
background: conic-gradient(at 50% -5%, #198072 0 , #11FACD 120deg, #00FA0A);
```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Rwm5ST9MXqmHNCln7C6fjyFAibfjJPOpPHRicPNssB7CNsp4ibibZEia3yjxqnyOuxZPia0naAmx7kOqdA/640?wx_fmt=png&from=appmsg)能得到混合的颜色效果，这对于随机性颜色背景来说十分有用。

