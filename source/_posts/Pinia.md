---
title: Pinia
cover_image: /img/cover.png
abbrlink: 74dce755
date: 2022-11-03 13:20:17
---

Pinia（菠萝）是Vue官方团队推荐替代Vuex的轻量级状态管理解决方案。设计理念是让Vue拥有Composition API 风格的状态管理库，支持Vue3的setup Composition API，并完整支持ts，探索 Vuex 的下一次迭代。

## 对比vuex

* 提供了更简单、符合组合式 API 风格的 API

* 弃用mutations；actions中可以异步操作或者同步修改state，可以作为常规的函数调用，而不是使用dispatch或者MapAction

* 完整的TS支持，友好的devTools支持

* 不再有嵌套结构的模块modules，支持多个stores，pinia提供的是一个扁平的stores结构，但仍然能够嵌套调用 stores 空间。

![](https://mmbiz.qpic.cn/mmbiz/rxLIic6e5g8RFZkn2cplFhCKRXGkwY5EicmddyUnnCRaIYyLQaj5cYj1JDtZpc1bnFh9N0ficF6vwWMcicWibfNMloA/640?wx_fmt=other)## stores

### Options API

```
// options storesexport const useCounterStore = defineStore("counter", ;  },  // 也可以这样定义  // state: () => ()  getters: ,  },  actions: ,  },});
```
### Composition API

对应关系：

* state--ref

* getter--computed

* action--functions

```
// setup storesimport  from "vue";export const useCounterStore = defineStore("counter", () => );  function increment()   return ;});
```
```
// 调用其它stores的数据export const useCounterStore = defineStore("counter", () => );  return ;});
```
## 在组件中使用

```
import  from './pinia/index.js';// 直接获取对象const counter = useCounterStore();let func = () => 
```
```
// 解构获取state, getters, actions// store是一个reactive包装的对象const  = storeToRefs(counter);// action 可以直接提取const  = counter;let func = () => 
```
```
counter.$patch();counter.$reset();
```
## mutations

在vuex中，我们通过提交 mutation 的方式，而非直接改变 store.state.count，是因为我们想要更明确地追踪到状态的变化。

此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。

### vuex为什么要区分mutations和actions

官方文档说明：“在 mutations 中混合异步调用会导致你的程序很难调试。例如，当你能调用了两个包含异步回调的 mutations 来改变状态，你怎么知道什么时候回调和哪个先回调呢？这就是为什么我们要区分这两个概念。在 Vuex 中，我们将全部的改变都用同步方式实现。我们将全部的异步操作都放在 Actions 中。”

事实上在 vuex 里面 actions 只是一个架构性的概念，并不是必须的，说到底只是一个函数，你在里面想干嘛都可以，只要最后触发 mutations 就行。vuex 真正限制你的只有 mutations 必须是同步的这一点（在 redux 里面就好像 reducer 必须同步返回下一个状态一样）。

同步的意义在于这样每一个 mutations 执行完成后都可以对应到一个新的状态（和 reducer 一样），这样 devtools 就可以打个 snapshot 存下来，然后就可以随便 time-travel 了。

如果你开着 devtool 调用一个异步的 actions，你可以清楚地看到它所调用的 mutations 是何时被记录下来的，并且可以立刻查看它们对应的状态。

---
所以只要能知道什么时候改变state （commit mutation） 然后去 snapshot 那个时候的 state，就能追踪到状态的变化，就不需要mutations，只需要保留一个用来异步操作的action.

---
https://www.zhihu.com/question/48759748/answer/112823337

https://segmentfault.com/q/1010000008640002

