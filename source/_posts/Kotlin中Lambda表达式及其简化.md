---
title: Kotlin中Lambda表达式及其简化
cover_image: /img/cover.png
abbrlink: 4f4fcc8b
date: 2022-11-13 12:54:47
---

在Kotlin编程中，lambda表达式的应用非常常见，这篇文章用来介绍一下怎样用lambda表达式定义函数，用lambda表达式进行函数传参，以及简化lambda表达式

参考文档: https://developer.android.com

## 用lambda表达式定义函数

#### 先来看一个最简单的例子：

```
val trick=
```
#### 关键点：

1.lambda表达式的结果是一个值，因此可以直接将其存储在变量中（即上图中的trick中）

2."="后面即为lambda表达式，大括号中的内容为函数正文

#### 怎样调用

```
fun main()
```
#### 加上参数和返回值

刚才的trick函数非常简单，它的数据类型为（）->Unit，（）为空，代表它不接受任何参数，返回值类型为Unit，代表它不返回任何内容，所以它较为完整的写法是这样的

```
val treat:()->Unit=
```
那么现在来看一个复杂的函数

```
val trick=fun trickOrTreat(isTrick:Boolean):()->Unitelse}
```
首先，我们又用lambda表达式定义了一个简单的函数trick，在trickOrTreat()函数的正文中，设置一个if语句，使其在 isTrick 为 true 时返回 trick() 函数，并在 isTrick 为 false 时返回 treat() 函数。

代码逻辑理好了，现在来测试一下：

```
fun main()
```
输出结果如下：

```
Have a treat!No treats!
```
在刚才的trickOrtreat函数中，我们将函数作为返回值，那么该怎样将函数作为参数传递呢？

举个栗子：

```
fun trickOrTreat(isTrick:Boolean,extraTreat:(Int)->String):()->Unitelse}
```
在这个函数中，添加了类型为(Int)->String的extraTreat函数参数，这代表trickOrTreat函数接受一个函数作为参数，其参数类型为Int返回值类型为String

那么现在我们来编写两个这样的函数来进行测试

```
fun main()//quantity为Int参数(可省略),lambda表达式不支持return语句，函数中最后一个表达式的结果将成为返回值，详情下回分解    val cupcake:(Int)->String=    val treatFunction = trickOrTreat(false，coins)    val trickFunction = trickOrTreat(true,cupcake)    treatFunction()    trickFunction()}
```
在main函数中，我们定义了两个以Int类型为参数，返回值为String的函数，并将其作为参数传入到trickOrTreat函数中。当 isTrick 为 false 时，传入 coins() 函数。对于第二次调用，当 isTrick 为 true 时，传入 cupcake() 函数。

好了，现在我们已经掌握了如何将函数作为参数。但是如果我们不想传入函数呢？答案是把函数类型声明为可为null,方法很简单，用圆括号括住函数类型，并在后面接?符号

修改trickOrTreat函数：

```
fun trickOrTreat(isTrick: Boolean, extraTreat: ((Int) -> String)?): () -> Unit  else         return treat    }}
```
好了，现在extraTreat可以为空了，并且只有它不为空的情况下才会调用println(extraTreat(5))

### 总结：

* Kotlin 中的函数是一级结构，可以视为数据类型。

* lambda 表达式提供了一种用于编写函数的简写语法。

* 您可以将函数类型传入其他函数。

* 对于一个函数类型，您可以从另一个函数返回它。

* lambda 表达式会返回最后一个表达式的值。

## 简化Lambda表达式

#### 1.省略参数名称

我们依然以上文中的coins函数为例

```
val coins: (Int) -> String = 
```
coins函数只接受一个Int类型的参数，我们完全可以省略quantity名称和->符号，因为·Kotlin会自动为其分配it名称，就像这样

```
val coins: (Int) -> String = 
```
#### 2.将lambda表达式直接传入函数

在上文中，我们将coins函数作为参数传入trickOrTreath函数中：

```
fun main()     val treatFunction = trickOrTreat(false, coins)}
```
我们可以看到，coins函数非常简短，并且只调用过一次，因此我们可以直接将lambda表达式传入其中，而不必再声明coins函数

```
fun main() )    treatFunction()}
```
#### 使用尾随lambda语法

当lambda表达式作为传入的组后一个参数时，可以把它写在括号外面，这样看起来会更加简洁

```
val treatFunction = trickOrTreat(false) 
```
#### 对比一下

最初代码：

```
fun main()     val treatFunction = trickOrTreat(false, coins)}
```
简化版：

```
fun main() }
```
这是lambda的一小步，是程序员的一大步

#### 高阶函数

最后再简单说一下高阶函数，其实非常简单，高阶函数能够返回或接受另一个函数作为参数，例如，上篇文章中介绍的trickOrTreat() 函数就是一个高阶函数的示例，因为它接受 ((Int) -> String)? 类型的函数作为参数，并且返回了 () -> Unit 类型的函数。

```
fun trickOrTreat(isTrick: Boolean, extraTreat: ((Int) -> String)?): () -> Unit  else         return treat//返回函数    }}
```
再来介绍一个非常有用的高阶函数：repeat

```
repeat(times: Int, action: (Int) -> Unit)
```
times 参数是操作应发生的次数。action 参数是一个接受单个 Int 参数并返回 Unit 类型的函数。action 函数的 Int 参数是到目前为止已执行的操作次数，例如第一次迭代的 0 参数或第二次迭代的 1 参数。您可以使用 repeat() 函数按指定次数重复执行代码，这与 for 循环类似。

使用示例：

```
fun main()     val trickFunction = trickOrTreat(true, null)    repeat(4)     trickFunction()}
```
运行后，treatFunction即可执行四次

#### 总结

* 只有一个参数的 lambda 表达式中，可以直接使用 it 标识符来引用它。
* 如果调用函数的最后一个参数是函数类型，可以使用尾随 lambda 语法将 lambda 表达式移至最后一个圆括号后面。
* 高阶函数是指接受函数作为参数或返回函数的函数。

