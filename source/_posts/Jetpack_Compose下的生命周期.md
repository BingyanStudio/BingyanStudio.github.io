---
title: Jetpack Compose下的生命周期
cover_image: /img/cover.png
abbrlink: f7e98eb0
date: 2024-03-10 15:05:05
---

# Jetpack Compose下的生命周期

当我们使用Jetpack Compose进行开发时，经常会遇到一些生命周期相关的问题，以下是一些生命周期相关问题的探索。

## ViewModel的生命周期：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8TICUU3A91FcGhBZoiaekAQLePjmSgTII3Jhc8iaOB3Fa879QM5Wk5HtW2jmTuR0eW79JKxpGj6Q6AQ/640?wx_fmt=png&from=appmsg)* ViewModel是一个用于存储和管理与UI相关的数据的类。它的生命周期与关联的Activity或Fragment不同。当Activity或Fragment被销毁时，ViewModel会继续存在，并且在新的Activity或Fragment实例创建时保持不变。

* ViewModel的生命周期是由Android系统管理的，它会在Activity或Fragment的创建和销毁过程中保持一致。这意味着ViewModel中的数据可以在Activity或Fragment的重建过程中被保留下来，而不会丢失。

* 若ViewModel是与Compose函数绑定。每个继承ViewModel的子类会在首次创建与之绑定的Compose时生成，在所有与之绑定的Compose函数出栈时销毁。

* 若是通过绑定获取，则使用的是依赖注入机理，安卓系统会保证每个ViewModel的子类仅存在一个对象。

* Example：

如下代码，MyViewModel仅与MyScreen进行绑定。当MyScreen首次创建时，MyViewModel会被创建，当MyScreen出栈时，它会被销毁。但当有其他Composable函数入栈时，MyScreen仍在栈中，因此MyViewModel不会销毁，仍会被保留。

但若是重新创建一个MyScreen，由于vm:MyViewModel=viewModel()使用的是依赖注入的机理，MyViewModel不会重新创建，而是调用唯一的一个MyViewModel

class MyViewModel:ViewModel(){}@Composablefun MyScreen()
## Composable函数的生命周期：

* Composable函数是一种用于构建UI界面的声明式编程模型。它的生命周期与Activity或Fragment的生命周期紧密相关。

* 当Activity或Fragment的生命周期发生变化时，Composable函数会根据需要进行重新计算和重绘UI界面。例如，当Activity或Fragment进入前台或后台时，Composable函数会相应地更新UI。

* Composable函数的生命周期是自动管理的，不需要我们手动处理。在调用时创建，在出栈时销毁。

* 值得注意的是，当Composable函数被其他Composable函数压栈时，不同于ViewModel，Composable函数会被销毁，只保留相关标识而不保留数据（最明显的特征是，当该Compose函数重新回到栈顶时，该Composable函数中的所有remember会被重新执行）

* Composable函数的刷新时机是：当在该函数中任何被引用的MutlableState类型的数据发生变化时。

* Example：

如以下代码，MainScreen会在创建时执行 a = remember，在a.intValue = 2被调用时，由于该被引用的MutlableState类型的数据发生变化，Composalbe函数会发生重组。

当调用nav.jump(AppRoute.MY_FEED)时，MainScreen函数会被FeedScreen压栈。此时FeedScreen进入栈顶，MainScreen函数会被销毁，只留标识符在栈里，数据被销毁。

当在FeedScreen中调用nav.pop()时，FeedScreen会出栈，MainScreen会重新进入栈顶。此时的MainScreen会重新创建。

例如，当你在MainScreen点击按钮，将a的值改为2后，跳转到FeedScreen，再跳转回来，a的值会重新执行remember而被重新赋值为1。

因此，若想跳转到其他界面再回来时，界面的数据不发生变化，就要把数据保留在ViewModel里而不是使用remember{}进行保留。

@Composablefun MainScreen(    vm : MainViewModel = viewModel(),    nav:NavHostController)    Button(onClick = )     nav.jump(AppRoute.MY_FEED)}@Composablefun FeedScreen(    vm:FeedViewModel= viewModel(),    nav: NavHostController)    nav.pop()}
## ViewModel的生命周期和Activity的生命周期的关系：

* ViewModel的生命周期比Activity的生命周期要长。当Activity被销毁时，ViewModel会仍然存在，并且可以在新的Activity实例创建时继续使用。这样可以确保ViewModel中的数据在Activity重建时得以保留。
* 通常情况下，一个Activity的生命周期可能会多次发生变化，例如从前台到后台，再到前台等。而ViewModel的生命周期只与Activity的创建和销毁相关，不受其他生命周期变化的影响。
## 在开发中遇到的生命周期问题

* 需要持久保留的状态使用ViewModel进行保存，推荐使用如下写法：

@Composablefun MyScreen(    vm : MyViewModel = viewModel())    a.intValue = 2}
* 注意Composable函数每次重组时，未remember的值都是深度创建（更改指针）

如以下代码，每次重组后，打印a的指针值，会是一样的，而打印b的指针值，会不同。

这种情况下尤其要注意回调的情况。因为在Kotlin里，回调使用的上下文的变量是将引用（指针）传入

@Composablefun MyScreen(    vm : MyViewModel = viewModel())    val b = MediaPlayer()    println(a)    println(b)}

