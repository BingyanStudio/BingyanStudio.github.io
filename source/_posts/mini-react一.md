---
title: mini-react（一）
cover_image: /img/cover.png
abbrlink: 4757500b
date: 2022-05-27 15:07:41
---

一个简单的 react 应用长这个样子：

```
import React from "react";import ReactDOM from "react-dom";import "./index.css";const hello = (  <div>    <h1>helloh1>    <h2>reacth2>  div>);ReactDOM.render(hello, document.getElementById("root"));
```
前几天刚更新的 react 18 不再推荐使用ReactDOM.render(hello, document.getElementById("root"));这种写法，但是由于我还没看过 react 18 都更新了啥，所以我们还是基于 react 17 来写 hhh

现在我们尝试来手动实现类似的效果

# 1. JSX

jsx 是 react 中的独特语法，粗略地讲的话：bable 将上面代码中的 hello 编译成 react 函数调用，该函数返回普通的 js 对象，然后 react 在 render 中调用浏览器的 api，递归地创建 DOM 元素。

例如，上面的 hello 会被编译为这样的 js：

```
const hello = React.createElement(  "div",  null,  React.createElement("h1", null, "hello"),  React.createElement("h2", null, "react"));
```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Scr88ggnib25xq6msOnASG1YNbudJ1QQD7zkZSUBbdSkxZmObiaica75lZYdzWmMw5SH5Wzj6N0YExA/640?wx_fmt=png)打印 hello 会出现如下对象：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Scr88ggnib25xq6msOnASG1p3u9QrUOg2QZz5tO3a6tQEtqsP9NHDQLW9T75fqGxWiaWvn63dGpOew/640?wx_fmt=png)其实 babel 对 jsx 的编译有新旧两种模式，旧版即编译为 React.createElement，但这就要求必须引入 React，新版则使用 react 17 新增的入口函数，具体见这里

在这里我们就先略过 babel 如何将 jsx 编译为 js，先来实现一个编译后使用的React.createElement

```
export type VDom =  |  & ;    }  | string;function createElement(  type: keyof HTMLElementTagNameMap,  props: Record<string, any>,  ...children: VDom[]): VDom ,  };}// 把我们自己的mini-react叫做Recat :Dconst Recat = ;
```
接下来是如何由 createElement 返回的对象创建 dom，也就是 render 函数的工作，实现一个 render 函数：

```
const Recat = ;const hello = Recat.createElement(  "div",  null,  Recat.createElement("h1", null, "hello"),  Recat.createElement("h2", null, "react"));function createDOMElement(vDom: VDom): HTMLElement | Text   const node = document.createElement(vDom.type);  for (const key in vDom.props) );    else if (key.startsWith("on"))  else node.setAttribute(key, vDom.props[key]);  }  return node;}function render(vDom: VDom, container: HTMLElement) Recat.render(hello, document.querySelector("#root"));
```
但是这样会有一个问题，上面的 render 操作无法中断，当 vDom 的结构很大很复杂的时候，会长时间的占用主线程进行 render，导致页面卡顿，这也就是 react 16 以前面临的问题，那如何解决这个问题呢

# 2. Fiber

先来介绍一些前置知识：

浏览器会 1 秒刷新 60 次，也就是近似每 16ms 刷新一次来保证用户所感受到的流畅性，而间隔的这 16ms 中又有一些时间是浏览器用来绘制内容的，所以交给 js 执行的时间大约只有 10ms 左右，如果 js 执行时间过长，用户就会感受到页面的卡顿。

像前面说的，上面的 render 一次性执行完成会阻塞主线程，那只要将这个任务分成一个又一个小部分就行，每部分执行完成后检查一下剩下的时间还够不够再执行一个任务，如果时间还够就继续执行，不然就将控制权交还给浏览器，等下一个空闲时间再执行。

思路有了，但是还要面临两个问题：

1. 如果还按照前面的数据结构，我们从中途暂停让出控制权后，是没有办法完成所有节点的渲染任务的
2. 如何寻找浏览器的剩余时间
对于第一个问题，react 构建了一种叫做 fiber 的数据结构，我们来对比一下这两种结构：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Scr88ggnib25xq6msOnASG1hdIZy0IJHrEwXgNUVyBibVRXIe1RicGicxvRk4qFqmzlyBb3jB7lnpPkA/640?wx_fmt=png)fiber使用 fiber 的结构，我们就可以在遍历的途中随时暂停，暂停时只要保存当前节点即可。

先定义一下 fiber 的结构

```
// 这和真实的fiber结构还是有很大出入的，在此为了方便我们将其修改为下面的简单结构// 对于普通文本节点也不能仅 string 了事，文本节点也需要有return等指向export const TEXT_ELEMENT = "TEXT_ELEMENT";export type Fiber =  | ((      |             | null;          child: Fiber;          stateNode: HTMLElement | null; // fiber对应的dom元素，create之前都是null        }      |     ) & )  | null;
```
然后需要将 vDom 转化成 fiber，这个问题我们留到下面跟着第二个问题一起解决

第一个问题解决了，第二个问题就更简单了，浏览器提供了这么一个 api，requestIdleCallback 接收一个回调函数，该回调函数会在浏览器空闲时执行。

React 实际使用的并不是这个接口，一方面是为了兼容性考虑，另一方面是这个 api 每 50ms 才会执行一次，远不能达到 react 的要求，react 最终是基于requestAnimationFrame实现了 Scheduler 来完成这个功能

继续修改之前的代码，由于代码量变大了，我稍微拆分一下文件，下面是 Recat 的主体部分：

```
import  from "./typings/Fiber";import  from "./typings/VDom";export const Recat = ;function createElement(  type: keyof HTMLElementTagNameMap,  props: Record<string, any>,  ...children: VDom[]): VDom ,  };}// createDOMElement不再深度create，只创建当前fiber的dom元素function createDOMElement(fiber: Fiber): HTMLElement | Text   const node = document.createElement(fiber.type);  for (const key in fiber.props || {})     node.setAttribute(key, fiber.props[key]);  }  return node;}// 下一个处理的fiber节点，render时对其进行初始化修改let nextUnitOfWork: Fiber = null;function render(vDom: VDom, container: HTMLElement) ,    return: null,    sibling: null,    stateNode: container,  };  // 开始创建节点  requestIdleCallback(workLoop);}function workLoop(deadline: IdleDeadline)   requestIdleCallback(workLoop); // 再次添加回调，等下一次浏览器空闲时执行}/** 负责完成当前任务，并返回下一任务 */function performUnitOfWork(fiber: Fiber | null): Fiber | null   if (fiber.return)   if (fiber.type !== "TEXT_ELEMENT")   let nextFiber = fiber;  while (nextFiber)   return null;}/** 将 fiber 节点的 children 进行 fiber 化 */function createChildrenFiber(fiber: Fiber) ;    } else ;    }    // 第 0 个作为父亲的 child，其他的则用 sibling 连接    if (!index) fiber.child = newFiber;    else preSibling.sibling = newFiber;    preSibling = newFiber;  });}
```
index.ts 中存放 vDom与 render 函数：

```
import  from "./recat";const hello = Recat.createElement(  "div",  null,  Recat.createElement("h1", null, "hello"),  Recat.createElement("h2", null, "react"));Recat.render(hello, document.querySelector("#root"));
```
# 3. render & commit

上面的代码能正常跑了，但是依然存在一些问题，在 performUnitOfWork 中，我们直接操作了 dom 元素，但是如果后面控制权被交出去执行了别的事，用户可能会看到渲染了一半的页面，现在需要解决这个问题。

React 是怎么操作的呢？

来扒一扒 react 的执行过程：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Scr88ggnib25xq6msOnASG1cIl03TB4TwA6Ff58Vkibic58QTVZwvWEqD86icCKQ8qA8LJicxlsKkOptw/640?wx_fmt=png)这是浏览器在绘制页面之前执行的操作。react 将修改 fiber 树与将修改绘制到页面上分成两个阶段，分别叫 render 与 commit，其中 render 阶段可以打断，commit 阶段不可打断。react 将 render 阶段负责计算的 fiber 树叫做 workInProgress tree，commit 阶段将 workInProgress tree 渲染成 dom 树。

修改之前的代码：

```
import  from "./typings/Fiber";import  from "./typings/VDom";export const Recat = ;function createElement(  type: keyof HTMLElementTagNameMap,  props: Record<string, any>,  ...children: VDom[]): VDom ,  };}// createDOMElement不再深度create，只创建当前fiber的dom元素function createDOMElement(fiber: Fiber): HTMLElement | Text   const node = document.createElement(fiber.type);  for (const key in fiber.props || {})     node.setAttribute(key, fiber.props[key]);  }  return node;}// 下一个处理的fiber节点，render时对其进行初始化修改let nextUnitOfWork: Fiber | null = null;// 需要渲染的树的根节点，render时进行初始化let wipRoot: Fiber = null;function render(vDom: VDom, container: HTMLElement) ,    return: null,    sibling: null,    stateNode: container,  };  wipRoot = nextUnitOfWork;  // 开始创建节点  requestIdleCallback(workLoop);}function workLoop(deadline: IdleDeadline)   if (!nextUnitOfWork && wipRoot) commitRoot();  requestIdleCallback(workLoop); // 再次添加回调，等下一次浏览器空闲时执行}/** 负责完成当前任务，并返回下一任务 */function performUnitOfWork(fiber: Fiber | null): Fiber | null   // if (fiber.return)   if (fiber.type !== "TEXT_ELEMENT")   let nextFiber = fiber;  while (nextFiber)   return null;}/** 将 fiber 节点的 children 进行 fiber 化 */function createChildrenFiber(fiber: Fiber) ;    } else ;    }    // 第 0 个作为父亲的 child，其他的则用 sibling 连接    if (!index) fiber.child = newFiber;    else preSibling.sibling = newFiber;    preSibling = newFiber;  });}function commitRoot() /** 递归操纵dom渲染页面，该过程不可打断 */function commitWork(fiber: Fiber) 
```
至此我们完成了页面挂载时候的行为，下面来看当页面发生更新时的情况

# 4. update

react 在发生更新时，会进行一遍 dom diff，比较即将要渲染的部分与上次渲染的部分，仅针对变化的地方发生修改。

更新的触发是通过再次调用 render 来实现的，在实际使用 react 时，我们只需要修改 state，react 会自动帮助我们调用 render，这部分我们放到后面来实现，现在我们先自己调用 render，用户端调用的代码如下：

```
import  from "./lib";let count = 0;function add() function minus() function Hello() , "+"),    Recat.createElement("button", , "-"),    // 由于我忘记处理文本节点可能是number的情况了，所以这里将就一下toString好了。。    Recat.createElement("h3", null, count.toString()),    Recat.createElement("input", null)  );  return hello;}Recat.render(Hello(), document.querySelector("#root"));
```
根据前面的代码，我们每次调用 render 时都会重新初始化 workInProgress tree，当触发更新调用 render 时，上次渲染的 fiber 树也就被覆盖了，而现在我们需要对比之前的 fiber 树与更新后的虚拟 DOM 树，然后生成一个记录了当前 fiber 节点如何变化的 fiber 树

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8Scr88ggnib25xq6msOnASG1krFyXbnfz6Uf4kuYEY38ScibUYjoZuAohvfnMydf1BASC5NC2eXqxjQ/640?wx_fmt=png)这一步骤发生在创建新 fiber 的过程里，也就是我们写的 createChildrenFiber，可以发现，diff 这一过程也是对 children 进行的，同时需要用到该父节点上一次渲染时的子 fiber，这就要求每一个节点都需要保存其上次渲染时的 fiber 节点。Fiber 类型中需增加新的属性 alternate

此外，节点有可能发生的更新变化有以下四种：

1. 新增

<div>  <div>1div>div><div>  <div>1div>  <div>2div>div>
2. 删除

<div>  <div>1div>  <div>2div>div><div>  <div>1div>div>
3. 更新，属性变化

<div>  <div>1div>  <div>2div>div><div>  <div>1div>  <div>3div>div>
4. 替换，标签 type 发生变化

<div>  <div>1div>  <div>2div>div><div>  <div>1div>  <span>2span>div>
基于上面两点，我们对 Fiber 类型做如下修改，新增两条属性：

```
// 这和真实的fiber结构还是有很大出入的，在此为了方便我们将其修改为下面的简单结构// 对于普通文本节点也不能仅 string 了事，文本节点也需要有return等指向export const TEXT_ELEMENT = "TEXT_ELEMENT";type EffectTag = "UPDATE" | "REPLACE" | "DELETE" | "CREATE" | "NO_EFFECT";export type Fiber =  | ((      |             | null;          child: Fiber;          stateNode: HTMLElement | null; // fiber对应的dom元素，create之前都是null        }      |     ) & )  | null;
```
💡 由于代码越来越长，我不想贴全部代码了，在这里放上这部分完成后的代码，下面仅记录修改的部分（不过之前都是在本地写忘记搞 git 仓库了所以没有前面的 commit 了 hhh

新增一个全局变量currentRoot，小小修改一下nextUnitOfWork的初始化

```
// 保存渲染前的fiber树let currentRoot: Fiber = null;export function render(vDom: VDom, container: HTMLElement) ,    return: null,    sibling: null,    stateNode: container,    alternate: currentRoot,    effectTag: "NO_EFFECT",  };  wipRoot = nextUnitOfWork;  // 开始创建节点  requestIdleCallback(workLoop);}
```
把前面的 createChildrenFiber 函数修改一下名字，改为 reconcileChildren，reconcile 详细说明可去React 官方文档看，懒得看也没关系，我们接下来要实现的就是这部分

```
/** 负责完成当前任务，并返回下一任务 */function performUnitOfWork(fiber: Fiber | null): Fiber | null   reconcileChildren(fiber);  if (fiber.type !== "TEXT_ELEMENT")   let nextFiber = fiber;  while (nextFiber)   return null;}/** 将 fiber 节点的 children 进行 fiber 化，同时对 children 进行 diff */function reconcileChildren(fiber: Fiber)   let preSibling: Fiber = null;  let index = 0;  while (index < curChildren.length || oldFiber)  else if (curVDom !== undefined && !oldFiber)  else if (curVDom === undefined && oldFiber)  else     if (oldFiber) oldFiber = oldFiber.sibling;    // 第 0 个作为父亲的 child，其他的则用 sibling 连接    if (!index) fiber.child = newFiber;    else preSibling.sibling = newFiber;    preSibling = newFiber;    index++;  }}
```
上面四个 getFiber 函数具体内容如下，其实又长没啥好看的。。以及，在 delete 时其实可以不用再生成一个新的 fiber 的，但那就需要在别的地方额外处理，由于我比较懒所以我还是多生成了一个节点

```
import  from "../typings/Fiber";import  from "../typings/VDom";export function getFiberInSameType(  curVDom: VDom,  oldFiber: Fiber,  returnFiber: Fiber)     newFiber = ;  } else ;  }  return newFiber;}export function getFiberInCreate(curVDom: VDom, returnFiber: Fiber) ;  } else ;  }  return newFiber;}export function getFiberInDelete(oldFiber: Fiber) ;  newFiber.effectTag = "DELETE";  return newFiber;}export function getFiberInReplace(  curVDom: VDom,  oldFiber: Fiber,  returnFiber: Fiber) ;  } else ;  }  return newFiber;}
```
在把 fiber 树提交上去进行渲染时，对不同的变化做不同处理

```
import  from "../typings/Fiber";import  from "./updateDom";/** 递归操纵dom渲染页面，该过程不可打断 */export function commitWork(fiber: Fiber)       break;    default:      break;  }  // if (fiber.return) fiber.return.stateNode.appendChild(fiber.stateNode);  if (fiber.type !== "TEXT_ELEMENT") commitWork(fiber.child);  commitWork(fiber.sibling);}
```
其中 update 时较为复杂，我们单独抽出来写：

```
export function updateDom(  dom: HTMLElement,  preProps: Record<string, any>,  curProps: Record<string, any>) 
```
至此 dom diff 与更新渲染完成（上面是没有 key 的情况，之后有时间我可能会补充有 key 情况下的 diff）

