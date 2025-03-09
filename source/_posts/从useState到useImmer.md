---
title: 从useState到useImmer
cover_image: /img/cover.png
abbrlink: 9f0c090b
date: 2024-03-28 05:39:14
---

在React开发中，组件的状态管理是我们处理最多的逻辑：

在一个组件之中，如果一个组件的某些数据是响应式的，会随着用户的点击，输入等交互逻辑而发生改变，这时我们通常会将这些数据放入组件里的状态（state）中来让react帮助我们管理，并期望在状态发生改变时组件能够重新渲染。

但是react追求数据的不变性来提高其性能。简单来说就是react在判断引用类型的state是否发生变化时，会依据在栈内存中的引用是否发生了改变来判断这个state是否发生了改变，而不会去比较这个对象内部的更深层引用是否发生更改。这就导致我们在更新组件的state变量的时候往往要将原来对象类型的state进行一次深拷贝生成一个新的对象，然后再去对新的对象进行修改，以达到更改引用的目的。

为了让大家更直观的理解这一原理，接下来我们将层层深入的分析不同类型的state在react代码中将如何进行修改。

```
function Switch()   return Light is ;}
```
首先是基本类型的state。

以上面的开关组件为例，isOpen是一个布尔类型的state变量，当我们向setIsopen传入!isOpen时，相当于是在用一个布尔类型的变量替代另一个布尔类型的变量，而这两个基本数据类型在栈内存中是不同的，react会发现这种不同，从而引发视图的更新。

接下来是引用类型的state

在一些表单中，我们将所有的数据都存放在一个对象中无疑是非常方便且符合直觉的。这种没有深层嵌套的对象仅仅使用ES6新语法中的展开运算符...进行拷贝就能完美的解决。

```
 function Person() );  function playWithHim() );    console.log("Happy! Happy! Happy!");    }  return (                             is  now !      );}
```
但是当我们要监听的值是有嵌套结构的对象时，似乎就没这么简单了。

```
function Person() ,    },    ,    },  ]);  function playWithHim(id)  };        else return ;      })    );    console.log("Happy! Happy! Happy!");  }  return (                    key=        >                       is             and                         ))}    
```
  );}在这种情况下，为了确保我们能够深拷贝整个对象，每多一层嵌套，就要多写一次...展开运算符。

看起来似乎不太优雅。

然而这种需要更新深层级的state的需求在实际的业务逻辑中是很常见的，比如带有多选功能的筛选列表，复杂的用户信息输入表单等。

通常情况下，我们可以通过调整数据结构，让数据扁平化来改善这种问题。

但是，如果你也和小🥣一样懒的话，也许Immer对你来说是一个不错的选择。

## 什么是Immer

下面是Immer官网对Immer的描述

> Immer（德语为：always）是一个小型包，可让您以更方便的方式使用不可变状态。
> 
> 接下来我们来看看Immer的这种方便体现在哪里。

## Immer解决的痛点

1. Immer 将不再需要创建对不可变对象进行深度更新时所需的典型样板代码：如果没有 Immer，则需要在每个级别手动制作对象副本。通常通过使用大量 ... 展开操作。使用 Immer 时，会对 draft 对象进行更改，该对象会记录更改并负责创建必要的副本，而不会影响原始对象。
2. 使用 Immer 时，您无需学习专用 API 或数据结构即可从范例中受益。使用 Immer，您将使用纯 JavaScript 数据结构，并使用众所周知的安全地可变 JavaScript API。
简单来说就是通过Immer，我们既可以确保不会更改原来的对象，同时也能够避开...展开的拷贝工作，且几乎不需要学习新的语法就能够轻松上手。

以useState和useImmer为例，来看看Immer为我们的代码简化了什么：

```
function Person() ,    },    ,    },  ]);  function playWithHim(id) );    console.log("Happy! Happy! Happy!");  }  return (                    key=        >                       is             and                         ))}      );}
```
可以看到，在setState函数中，我们直接对变量draft进行修改，这不仅仅避开了使用对原来的对象的深拷贝，也使代码更加清晰可读，逻辑一目了然。

## 背后的原理

> Immer就像是一个你的助理，他给你一封信和一份副本草稿，这个草稿来记录你希望在信上进行的修改。等你的所有修改都完成后，通知这位助手来收回草稿以生成你的下一封信。
> 
> Immer的原理其实非常简单，Immer利用 produce 函数，它将我们要更改的 state 作为第一个参数，对于第二个参数，我们传递一个名为 recipe 的函数，该函数传递一个 draft 参数，我们可以对其应用直接的 mutations。一旦 recipe 执行完成，这些 mutations 被记录并用于产生下一个状态。produce 将负责所有必要的复制，并通过冻结数据来防止未来的意外修改。

参考：

Immer中文文档

React官方文档

React中为什么要强调使用Immutable - 知乎 (zhihu.com)

