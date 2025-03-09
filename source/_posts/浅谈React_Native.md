---
title: 浅谈React Native
cover_image: /img/cover.png
abbrlink: 5b06f6f9
date: 2024-03-14 12:11:01
---

# 

> React Native （后面简称 RN）是基于 JavaScript 语言和 React 框架的移动端跨端方案。不同于 WebView 容器，RN 所编写的应用具有 Native 平台的特性，而不是简单的 Web 的 “套壳”。不多说了，原生，启动！
> 
> ## 为什么选择 RN

因为我可以愉快的写 JS😋

移动跨端方案目前比较火的就是 RN 和 Flutter 两个框架，前者是 FaceBook (现在的 Meta)在 2013 的一个内部的 Hackson 项目中诞生的，而后者则是 Google 在 2015 年发布的。

那在这两个框架面前应该如何选择呢？

在不少的论坛帖子下能看到诸如 “都 2023 年了还学 RN，赶紧入 Flutter” 这样的发言。那么 Flutter 一定就比 RN 更好吗？其实也未必。编程没有所谓的银弹，所有的跨端方案都不是完美的，所以需要根据自己的实际情况和项目需求选择一套适合的框架。

对于学过前端的同学来说，RN 的上手难度肯定是更低的，在已经会 JS 和 React 的基础上，再去了解一些 RN 的 “村规” 基本上很快就能开始写东西了。而 Flutter 则是用 dart 语言来写的，虽然说语法和 js 比较像，但终究是两门不一样的编程语言，学起来还是有成本的。

既然选择了 RN，接下来就谈谈 RN 的一些特性。

### 1.生态丰富

RN 从 2013 年就已经存在了，发展到现在积累的生态是相当可观的。比如让项目更加工程化的 Expo 框架，基于 RN 开发了很多好用的 API，路由管理和错误处理，再比如专门用于做动画的 Reanimated 库等等。

### 2.新架构

新架构是在 RN 0.68 之后正式开始测试的（到我用的时候最新的版本是 RN 0.72），RN 在新架构之前在性能上饱受诟病，在新架构推出后，引入了 JSI, Fabric 等新的模块，使得 RN 启动性能、通信性能都有明显提升，渲染流程也有了很大的变化，具体机制后面会简单提到。

下面具体说说新架构，来了解 RN 是怎么运作的。

## RN 新架构

这是一张 RN 新老架构的对比图，不论是老架构还是新架构，都需要一个地方来运行 JS 代码，一个地方进行原生视图的绘制。这里就会涉及到 RN 的线程相关的概念。

RN 主要的线程有三个：

1. 主线程（也叫做 UI 线程）
2. JavaScript 线程
3. Background 线程（也叫做 ShadowQueue 线程）
主线程负责所有的视图操作，也是所有 Native 代码运行的地方，除了主线程外的线程是没有直接操作视图的能力的。

JS 线程顾名思义就是用来运行 JS 的，一些用户的交互事件，发请求收请求，React 相关的逻辑会在这里执行

还有一个 Background 线程，主要用于布局，会交由 Yoga 布局引擎来执行。

### 旧架构

有了前面的铺垫，旧架构下的运行逻辑就可以摸索出来了。

JS 代码会运行在 JS 线程，并交给 JSC (JavaScript Core) 引擎来执行。它与 UI 线程的通讯则需要通过 JS Bridge 来实现，JS Bridge 需要对通讯消息进行序列化和反序列化，从而实现 JS 语言与 C++ 的通讯。消息到达主线程后，就会调用 C++ 的 API ，进行布局的计算，最后通过 Java/OC 对操作系统提供的原生组件进行处理，调用渲染引擎把 UI 绘制出来。

### 新架构

新架构带来的变化中，有挺多值得展开讲讲的点，但是受限于篇幅，我只想分享一下新架构引入的新 JS 引擎 Hermes **以及 新的渲染器 Fabric。

Hermes

Hermes 引擎是 Facebook 专门为移动端优化过的 JS 引擎，其他 JS 引擎比如 V8 (Chrome、Node.js)，JSC （旧版的 RN、Safari）多少是从桌面时代就存在的，所以难免会有一些 “历史包袱”。

旧版架构中的 JSC 引擎采用了 JIT 即时编译，运行过程大概是编译一段执行一段，但是在初始化的时候，JSC 引擎需要把整个 JS 代码都编译和执行一次（为了后续 JIT 的编译和优化）。这样一来显然启动速度就会变慢。

而 Hermes 则是会在本地先将 JS 编译为字节码，然后再下发字节码，这就是为什么 Hermes 会提高首屏性能，大概是原来的 2 倍左右

Frabric

Frabic 是新架构引入的渲染器，在 Frabric 中使用了 JSI (JavaScript Interface) ，用来替换之前的 JS Bridge，JSI 能让 JS 直接调用 C++，而无需经历序列化和反序列化的转化，这样一来效率就变高了，大概是原来的 3 倍左右。此外，Frabic 还重写了 Java/OC 层的大多数 Native 组件，性能得到了比较可观的提升。

至此架构的分享就差不多了，至于没有提及的 Turbo Modules 部分，以及其他更详细的细节，就请自己探索吧！

## 实现一个简易瀑布流效果

这是我刚开始学 RN 的时候写的一个小 demo，也正是这个小 demo 让我体会到了写 web 和写 RN 之间的区别。

想要实现瀑布流，首先就要选择能展示图片的组件，RN 中这个组件是  ， 组件最常用来展示 url 指向的图片资源，但是，比较坑的一点就是，如果不给定   一个具体的长和宽，则图片是显示不出来的，这一点在 RN 的文档里有提到。

所以我们必须给每个图片一个明确的宽高，但是我们既然要瀑布流效果，那图片的高度必然是错落有致的，也就是说，图片的高度应该根据图片的真实高度计算而来，并且赋值给  的高度样式。

我写了一篇文章专门总结了如何实现 Image 自动高度，如果不知道的话可以看看这个文章。

然后是样式的添加，在 RN 中默认是不能使用 CSS 的，我们得用 StyleSheet  创建一个对象，并且用内联的形式把样式传递给每个需要的组件。

此外，我们还需要一个能滚动的视图。这在 Web 是不难实现的，只需要指定父容器的 overflow 属性为 auto 或者 scroll 即可，但是在 RN 中，普通的  ****没有滚动视图的能力的，需要用  （也可用  等），完整的组件代码如下，这里实现了一个简单的双列瀑布，左边是固定高度，右边是自动高度。

```
import React,  from "react";import  from "react-native";const leftImages = [  ,  ,  ,  ,  ,  ,  ,  ,];const rightImages = [  ,  ,  ,  ,  ,  ,  ,  ,];export function ImageWaterfall() );        },        (error) =>       );    });  };  useEffect(() =>  = await getImageSize(image.url);            return (              <Image                key=                source=}                alt=""                style=]}              />            );          } catch (error)         })      );      setRightImageComponents(components);    };    loadImages();  }, [rightImageComponents]);  return (    <ScrollView>      <View style=>        <View style=>                        source=}              alt=""              style=            />          ))}        View>        <View style=>View>      View>    ScrollView>  );}const styles = StyleSheet.create(,  image: ,  imagesColumn: ,  leftColumn: ,  leftImage: ,});
```
💡 这只是一个最简单的例子，一个完整的瀑布流，还需要图片的 “无限” 加载，懒加载等处理，可以自己动手试试！

