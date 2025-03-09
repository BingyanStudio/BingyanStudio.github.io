---
title: 浅谈JS函数式编程
cover_image: /img/cover.png
abbrlink: 8936a97a
date: 2022-12-01 15:02:15
---

### 高效搬砖——拆分、组合、复用

我们不妨以一个栗子开始：这是Scott Sauyet所著《Favoring Curry》中一个在项目实战里遇到的问题。

```
var data = ,        ,        ,        ,        ,            ]};
```
这是一段服务器返回的（parse过的）JSON数据，现在要求是，找到用户Scott的所有未完成任务，并按到期日期升序排列。如果我们用命令式代码来写，大概是这样的画风：

```
getIncompleteTaskSummaries = function(membername) )        .then(function(tasks)             }            return results;        })        .then(function(tasks)             }            return results;        })        .then(function(tasks) )            }            return results;        })        .then(function(tasks) );            return tasks;        });};
```
可能你和我一样，看到这段代码的第一反应是：诶呀好复杂>_<感觉脑子转不过来了（翻译：这个人是用脚写的代码？）。也有可能，你天赋异禀，可以一瞬间理解这个函数每一步都是在干什么。还有可能的是，你也写过这样的代码。这都没关系，但我们至少可以达成一个共识：这段代码是存在问题的。

那么，问题出在哪里？

其一，我们注意到，getIncompleteTaskSummaries是一个具有较高抽象层次的函数。如果是我来实现，会遵循这样的原则：

高层次抽象低层次抽象底层实现而这里，整个函数体都充满了底层实现。也就是说，上述代码缺失了低层次抽象这个环节。其实，所有调用在then方法中传入的回调其实都可以单独取个名字，然后拿出来复用。

甚至，我们可以把一个回调再拆成多个可复用的函数，这样就有更大的灵活性。比如：最容易想到的是，循环这个结构，一共使用了三次，我们能否把循环也做成一个函数呢？

其二，getIncompleteTaskSummaries这个函数本身缺乏复用性，如果我把需求从“未完成任务”改成了“已完成任务”，或者把“升序”改为“降序”，就会导致整个getIncompleteTaskSummaries重写。

以上提到的两点，违背了程序设计中的同一个原则：DRY（Don't Repeat Yourself)

其三，如果你是一个和我一样的普通人，恐怕无法在10秒内读懂这个getIncompleteTaskSummaries到底是在干什么（虽然它取了个这么长的名字，但光看名字还是语焉不详）。这段代码不易理解，归根结底是因为它每一步都缺乏语义化的说明，因此更接近计算机语言而非自然语言。换言之，它是命令式的，而非声明式的。

这又违背了程序设计中的一个基本原则：KISS（Keep It Simple, Stupid）

那么，要怎么做，才能更优雅地搬砖呢？其实根据上面的讨论，不难看出，我们需要这样一种编程范式，它具有如下特性：

* 抽象的目标是过程，抽象的工具是函数
* 有分明的层次，高层次抽象的函数由低层次抽象的函数组合起来
* 声明式
这就是函数式编程的基本思想。带着这些思想，我们来看看我们要做哪些事。

* 从data中取出tasks
* 筛选出username为Scott，且complete为false的对象
* 循环tasks数组
* 对每个元素判断是否为真，并加入新数组
* 返回新数组
* 根据dueDate升序排序
这其中每一步都是我之前提到的低层次抽象。

进一步观察，我们发现第二步是可以进一步推广的，因为我们不仅需要一个“筛选出username为Scott，且complete为false的对象”的函数，很可能我们之后会多次用到一个“筛选出满足一个布尔表达式为真的元素”的函数。而这就是ES6中filter的来历。

再进一步，我们发现在实现filter函数的时候会用到循环，而正如我之前提到的，循环是必然会被无数次重用的，那就把循环也写成函数吧！也即，forEach的来历。

第三步中也有一些可说的，比如“根据dueDate升序排序”也可以被推广为“根据任意属性升序排序”，产生一个可复用的函数sortBy。

```
function at (obj, prop) function atTasks (obj) function forEach (arr, callback) function isIncomplete (obj) function isAssignedTo (username) }function satisfy (...restrictions) }function filter (arr, callback) )    return res}function sortBy (arr, prop) )}function sortByDueDate (arr) function getIncompleteTaskSummaries(membername) )}
```
可以看到，这样一段代码是有着无限扩展性和灵活性的。完美解决了第一和第二个问题。

再看第三个问题。如果接手这段代码的人（或者写完这段代码一个月后的我）想要看懂整个程序在干什么，那么只需要看最后的函数体就可以了，而最后的函数体已经接近自然语言，在十秒内理解完全不成问题。另一方面，如果ta想要了解其中一个函数的底层实现，那么可以跳读到对应的函数体，注意到，最长的函数体不超过五行。至此，第三个问题也得到了解决。

但是，经过分析，我发现这样的代码依然美中不足。

其一，如at，forEach，filter，sortBy这样的函数，不属于业务逻辑，而且不止我这个项目中需要，而是应用非常广泛的util，这时我会想，如果有一个现成的库已经实现了这些轮子，那就好了。

其二，虽然最后一行“已经接近自然语言”，但假设我只有一颗仓鼠脑，觉得杂乱无章的括号还是妨碍了我理解，怎么办？其实，就像我之前列的表一样，要做的事情其实无非是：按顺序完成一系列任务，然后把它们连缀起来。进一步说，就是实现一个函数，使得：

其三，为了语义通顺，我定义了很多名字很长但是不能复用的函数，如atTasks，sortByDueDate等，造成了命名空间的浪费甚至污染。既然这些函数都是一次性的，如果我们能够写成at('tasks')()或者sortBy('dueDate')()这样的柯里化函数，就可以在不定义新函数的前提下达到相同的效果，但如果我们用闭包进行手动柯里化未免过于麻烦了。

正好有这样一个库lodash.js为我们改进了这三点。我们引入lodash/fp模块，将以上代码改写如下。

```
const _ = require('lodash/fp')const at = _.curry(function (prop, obj) )const isIncomplete = function (obj) const isAssignedTo = _.curry(function (username, obj) )const satisfy = function (...restrictions) }const tasksIncompleteAndAssignedToScottSortedByDueDate =       _.flow(       at('tasks'),        _.filter(satisfy(isIncomplete, isAssignedTo('Scott'))),        _.sortBy('dueDate')      )const getIncompleteTaskSummaries = function () 
```
这样的代码，即使是从来没学过编程的人也可以大致看懂它在做什么。甚至可以说，阅读这样的代码是一件极度舒适（bushi）的事情。

### 优雅的异常处理——Maybe函子

在处理服务器返回的数据时，有可能会遇到空值null，当访问null的属性时会引起Javascript错误。这就意味着代码中必须充斥大量判空语句，干扰业务逻辑，并且严重降低可读性。下面只是一个简单的示例，实际情况可能会复杂得多。

```
function getCountry(student)     }    return 'Country does not exist'}
```
可以看到，随着if嵌套加深，代码可读性会大幅度下降。

为解决这一问题，我们引入一种叫做Maybe函子的结构，你可以把Maybe理解为一个包裹着值的容器。Maybe分为两种，Just和Nothing。Just类和Nothing类都实现了map方法，Just类的map方法接受一个函数f，会执行这样的操作：将值val从容器中拿出来，计算得到f(val)，返回一个包裹着f(val)的新容器（这是为了隔离副作用）。如果f(val)不为空，则新容器类型也为Just，否则为Nothing。Nothing类的map方法只会返回Nothing容器本身。

上面说得可能还是略抽象，但其实代码很简短。以下是一个Maybe的简易实现。

```
const isInvalid = function (val) class Maybe }class Just extends Maybe   map (f)   getOr () }class Nothing extends Maybe   getOr (other) }
```
现在使用Maybe这一结构，重写之前的getCountry函数。

```
const getCountry = function (student) 
```
可以看到，Maybe的原理是：静默处理空值，不打断链式调用，把错误统一留到最后处理。

这里仅仅介绍了用Maybe判空的方法。如果要处理其他错误，其实只要改isInvalid函数即可。

### 无懈可击——单元测试

单元测试，顾名思义，就是把大项目拆成一个个单元，分别测试它们的功能。这些单元各自具有相对独立的功能，而且与外部代码隔离。一般来说，一个单元就是一个函数。

可以想见，如果对命令式代码进行单元测试，会举步维艰。原因主要是：

* 不可重复性：因为命令式代码依赖大量共享状态，如果一次单元测试改变了共享状态，重复测试结果就会不同。（最简单的例子：给计数器+1后返回）

* 不满足交换律：Clean Code的作者Robert C. Martin在一次演讲中曾问过观众这样一个问题：“你们有没有这样的经历？程序报错了，你百思不得其解，机缘巧合，你改变了两个函数的运行顺序，发现它居然跑起来了，但你至今都不知道为什么。”台下有过半的人都举起了手。这也是因为：一个函数f1需要依赖另一个函数f2修改共享状态后才能执行，交换运行顺序会导致不可预测的后果。

反过来，函数式编程则完全为单元测试铺平了道路。纯函数不依赖外部状态、无副作用的特性使得一个函数就是一个沙箱。

下面，简单介绍一下借助Qunit进行单元测试的方法。

```
$ npm i qunit -D
```
在项目根目录建立如下文件夹：

```
$ lsnode_modules/ package-lock.json package.json script/ test/
```
在script目录下编写源文件（这里假设文件名为index.js）：

```
const add = function (a, b) module.exports = 
```
在test目录下编写测试文件：

```
const MyModule = require('../script/index')QUnit.module('MyModule')QUnit.test('test add', assert => )
```
回到项目根目录，在package.json中添加一项scripts：

```
,  "scripts": }
```
打开命令行，输入测试命令：

```
$ npm test
```
结果是not ok，得到详细信息如下：

```
> test> qunitTAP version 13not ok 1 MyModule > test add  ---  message: failed  severity: failed  actual  : 3  expected: 19260817  stack: |        at Object. (C:\Users\Leslie\Desktop\test\qunit\test\test.js:4:10)  ...1..1# pass 0# skip 0# todo 0# fail 1
```
参考文献：

[1]: Functional Programming in Javascript (2016), Luis Atencio

[2]: Favoring Curry, Scott Sauyet

[3]: Pointfree编程风格指南, 阮一峰

