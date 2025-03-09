---
title: preact 介绍
cover_image: /img/cover.png
abbrlink: 81174ae
date: 2022-10-21 12:33:58
---

## preact 是什么

> Fast 3kB alternative to React with the same modern API
> 
> 我的理解: preact 是一个精简(无论是体积还是整体设计)版的 react

官方文档: https://preactjs.com/

## preact 和 react 的关系

他和 react 是两个团队做的，两个侧重点不同的库

react 是一个大而全的库，他在性能优化、社区建设等方向有着巨大的领先优势但是 preact 走的是极简路线，虽然他性能上没 react 好，边界情况没 react 考虑周全，但是他真的太小了，所以有了两个优势：

1. 引入代价极小，只有 3kb 大小
2. 学习成本更低，react 源码没有人指导根本看不懂，preact 不一样，每个文件都小小的，逻辑清晰，一看就会
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RJNDfRbv186KxFj5sTLzvSDTsCRhUkoPVynBY9MJayDXLy1viaAnmbicouwhibKOVr0Ts8ndEOw7iaXQ/640?wx_fmt=png)react_linecreact 核心模块代码量 👆

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RJNDfRbv186KxFj5sTLzvS2rK5ibK1EnwFpqV6UKMiaNrsayTzkn34OHsqs8fVaS5GzYibW8ibhf6Uiaw/640?wx_fmt=png)preact_linecpreact 核心模块代码量 👆

所以学习 preact 一个很重要的点是方便我们理解 react 的核心思想，即常说的 虚拟 dom、dom diff、hook 等但是从 react 16 起的一些新特性，诸如 fiber、suspense mode、concurrent mode 等，preact 出于简洁考虑就没有支持，所以 除了通过 preact 学习上述共通部分，还需要自己了解 react 16 之后的新特性

### 二者的区别

上文说到两个库有共通部分，这里指出一些差别，避免误导大家

| 功能点 | preact | react |
| --- | --- | --- |
| dom diff | 对 key 的处理较为粗糙 | 考虑了更多边界情况 |
| 点击事件 | 对齐原生 | 使用合成事件、事件代理 |
| hook | 数组 | 链表 |

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RJNDfRbv186KxFj5sTLzvSaZ4E6ibMQSSLSibP2wF22a1UHMjme2ujcNlJMYu9VrAdxXRrtSsuMcuA/640?wx_fmt=png)react_hook_listreact hook 模块代码 👆

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RJNDfRbv186KxFj5sTLzvSmHM4YkRnvZG7gJg4HK3nqknJ3Jql58xb63OMCKLgN8ib9zag1dyaWbw/640?wx_fmt=png)preact_hook_listpreact hook 模块代码 👆

### preact 有而 react 没有的

#### signals

文档：https://preactjs.com/guide/v10/signals

signal 是一个独立于 preact 的库，他的写法类似 vue 里的 reactive 模块， 而且二者都是独立于框架的，也就是你可以在 react 里用 signal，在 preact 里用 vue/reactive。值得学习的是他的思想

react hook 的写法写个计数器👇

```
import  from "react";function Counter()   return (          Count: 

      click me      );}
```
preact signal 的写法写个计数器👇

```
import  from "@preact/signals";// Create a signal that can be subscribed to:const count = signal(0);function Counter()   return (          Count: 

      click me      );}
```
虽然这样看两种写法差的不多，但是你想想，最主要的区别在哪里？useState hook 只能在 react 组件内用，不能库组件传输数据 （如果要这么做得依赖别的方法，比如提升到父组件再传下去等等），但是 signal 可以不在组件内！直接实现了跨组件传输数据！

我们知道，调用 setState 可以触发 react 重新渲染，但是为啥count.value++;就可以触发界面更新呢？这里的主要点在于 对 value 熟悉的 set 操作做拦截

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RJNDfRbv186KxFj5sTLzvSqFvKicZTC9DmO8PicbficuaFyAZ6raLcSymnutBGyibiazD4CiatwhxtibbPw/640?wx_fmt=png)preact_signal_setValue具体细节感兴趣可以自己找文章看看，这里不赘述

此外，preact 为了支持 signal 也蛮拼的，支持 preact 倒是好说，preact 暴露了一些全局的钩子配置，可以很方便的侵入渲染逻辑

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RJNDfRbv186KxFj5sTLzvSmaMvVlQ3662ZJsTCXqx1qZVJV4pDfe9Cicq65nFPh94xzibtmBrLbRNw/640?wx_fmt=png)preact_hook_preact但是 react 没有对外提供直接的接口，想修改他的渲染逻辑只能...

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RJNDfRbv186KxFj5sTLzvSNLoXRH1uGJBNe72Xs3SykgKFxzP8cU9qCEwCPnSuOic9zcAk5SQmKbQ/640?wx_fmt=png)preact_hook_react#### web component

由于没有使用 react 那样的合成事件，preact 和原生贴合的做法让他直接就能支持 web component，这里不多说明，贴个官方文档：https://preactjs.com/guide/v10/web-components

## 补充一些 tips

* 看对比文章的时候看最新的。网上搜 react preact 对比，有很多 17 年的文章就别看了
* 学习 react 源码可以先看 preact。react 源码结构过于复杂，你根本找不到从哪里开始看
* 看文档最好看源语言文档。翻译的文档存在滞后的问题（如写本文的前几天 preact 官方文档的中文版本还没有对 signal 的介绍。不是没翻译，是压根没有）

