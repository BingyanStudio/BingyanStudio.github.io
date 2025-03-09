---
title: 初步探索Android中的依赖项注入
cover_image: /img/cover.png
abbrlink: b09b411
date: 2023-10-15 13:13:40
---

# 

## 什么是依赖项注入

类通常需要引用其他类。例如，Car 类可能需要引用 Engine 类。这些必需类称为依赖项，在此示例中，Car 类依赖于拥有 Engine 类的一个实例才能运行

类可通过以下三种方式获取所需的对象：

1. 类构造其所需的依赖项。在以上示例中，Car 将创建并初始化自己的 Engine 实例。
2. 从其他地方抓取。某些 Android API（如 Context getter 和 getSystemService()）的工作原理便是如此。
3. 以参数形式提供。应用可以在构造类时提供这些依赖项，或者将这些依赖项传入需要各个依赖项的函数。在以上示例中，Car 构造函数将接收 Engine 作为参数。
第三种方式就是依赖项注入！使用这种方法，我们可以获取并提供类的依赖项，而不必让类实例自行获取。

下面是一个示例。在不使用依赖项注入的情况下，要表示 Car 创建自己的 Engine 依赖项，代码如下所示：

```
open class Engine()}class Car }fun main() 
```
这并非依赖项注入的示例，因为 Car 类构造了自己的 Engine。这可能会有问题，原因如下：

* Car 和 Engine 密切相关 - Car 的实例使用一种类型的 Engine，并且无法轻松使用子类或替代实现。例如：

我想要制作一款用汽油引擎的车和一款用电动引擎的车，那么我便需要这样做：

```
open class Engine()}class GasEngine():Engine()}class ElectricEngine():Engine()}class Car1()}class Car2()}fun main()
```
如代码所示，我需要制作两种Car，Car1会在自己的实例中创造一个GasEngine实例，Car2会在自己的实例中创造一个ElectricEngine实例。

这样如果我们每换一种引擎，就需要新制作一种车，除了更换引擎外需要将其他配件原封不动复制过去。如果我们有100种引擎，就需要制作Car100  ctrl c/v大法会助力代码变成一坨

这样是违背常识的，引擎作为配件，理应是可以替代的，例如一种车应该存在汽油引擎版和电动引擎版。

如果使用依赖项注入，那会是什么样子的呢？如下：

```
open class Engine()}class GasEngine():Engine()}class GasEngine():Engine()}class ElectricEngine():Engine()}fun main()
```
我们只制作了一种车，它接收一个“engine”参数作为引擎配件

当我们在main函数里，使用GasEngine的实例注入Car时，便能制作一辆汽油引擎的车car1，使用ElectricEngine的实例注入Car时，便能制作一辆电动引擎的车car2，这样我们就不用再定义Car1、Car2...CarN了。

main 函数使用 Car。由于 Car 依赖于 Engine，因此应用会创建 Engine 的实例，然后使用它构造 Car 的实例。

Android 中有两种主要的依赖项注入方式：

* 构造函数注入。这就是上面描述的方式。我们将某个类的依赖项传入其构造函数。
* 字段注入（或 setter 注入）。某些 Android 框架类（如 activity 和 fragment）由系统实例化，因此无法进行构造函数注入。使用字段注入时，依赖项将在创建类后实例化。代码如下所示：
```
class Car }fun main(args: Array) 
```
在这里，我们必须要立即开工制作一辆车，但是这种车使用什么样的引擎我们仍然悬而未决，因而我们将引擎设置为延迟初始化（lateinit var），即待定。

在经过一番激烈的讨论后，我们决定好了引擎装什么，便使用car.engine = Engine()将决定好的Engine实例装到car的引擎上。

自动依赖项注入

在如上示例中，我们自行创建、提供并管理不同类的依赖项，而不依赖于库。这称为手动依赖项注入或人工依赖项注入。在 Car 示例中，只有一个依赖项，但依赖项和类越多，手动依赖项注入就越繁琐。手动依赖项注入还会带来多个问题：

* 对于大型应用，获取所有依赖项并正确连接它们可能需要大量样板代码。在多层架构中，要为顶层创建一个对象，必须提供其下层的所有依赖项。例如，要制造一辆真车，可能需要引擎、传动装置、底盘以及其他部件；而要制造引擎，则需要汽缸和火花塞。
* 如果我们无法在传入依赖项之前构造依赖项（例如，当使用延迟初始化或将对象作用域限定为应用流时），则需要编写并维护管理内存中依赖项生命周期的自定义容器（或依赖关系图）。
以下我将介绍自动依赖项注入的一种：使用Hilt实现依赖项注入

## 使用 Hilt 实现依赖项注入初步

### 添加依赖项

首先，将 hilt-android-gradle-plugin 插件添加到项目的根级 build.gradle 文件中：

```
plugins 
```
然后，应用 Gradle 插件并在 app/build.gradle 文件中添加以下依赖项：

```
plugins android dependencies // Allow references to generated codekapt 
```
Hilt 使用 Java 8 功能[1]。如需在项目中启用 Java 8，请将以下代码添加到 app/build.gradle 文件中：

### Hilt 应用类

所有使用 Hilt 的应用都必须包含一个带有 @HiltAndroidApp 注解的 Application[2] 类。

@HiltAndroidApp 会触发 Hilt 的代码生成操作，生成的代码包括应用的一个基类，该基类充当应用级依赖项容器。

```
@HiltAndroidAppclass ExampleApplication : Application() 
```
生成的这一 Hilt 组件会附加到 Application 对象的生命周期，并为其提供依赖项。此外，它也是应用的父组件，这意味着，其他组件可以访问它提供的依赖项。

### 将依赖项注入Andriod类

在 Application 类中设置了 Hilt 且有了应用级组件后，Hilt 可以为带有 @AndroidEntryPoint 注解的其他 Android 类提供依赖项：

```
@AndroidEntryPointclass ExampleActivity : AppCompatActivity() 
```
Hilt 目前支持以下 Android 类：

* Application（通过使用 @HiltAndroidApp）
* ViewModel（通过使用 @HiltViewModel）
* Activity
* Fragment
* View
* Service
* BroadcastReceiver
如果我们使用 @AndroidEntryPoint 为某个 Android 类添加注解，则还必须为依赖于该类的 Android 类添加注解。例如，如果我们为某个 fragment 添加注解，则还必须为使用该 fragment 的所有 activity 添加注解。

@AndroidEntryPoint 会为项目中的每个 Android 类生成一个单独的 Hilt 组件。这些组件可以从它们各自的父类接收依赖项，如组件层次结构种所述

此外，还有一些其他注解，例如：

* 使用@Inject注解来标记需要注入的依赖项
* 使用@Module注解来标记用于提供依赖项的模块类
* 使用@Provider注解来标记提供依赖项的方法等
在了解以上这些知识后，让我们使用Hilt在以安卓下实现前面造车的例子吧（以Compose为例）

### Hilt注入依赖项的实践

请先看以下代码：

```
class Tire()}class Horn()}class Bulb()}open class Engine @Inject constructor()}class GasEngine @Inject constructor() : Engine()}class ElectricEngine @Inject constructor() : Engine()}class Car @Inject constructor(    private val engine: Engine,    private val tire: Tire,    private val horn: Horn,    private val bulb: Bulb)    fun check()}object CarStore@Module@InstallIn(SingletonComponent::class)object CarFactory }
```
代码很长，让我们来分部分分析

```
class Tire()}class Horn()}class Bulb()}open class Engine @Inject constructor()}class GasEngine @Inject constructor() : Engine()}class ElectricEngine @Inject constructor() : Engine()}
```
造一个车需要许多零件，这里我们以轮胎、喇叭、车灯、引擎为例，前三者均需要在车开启前检查，而引擎可以开启，且引擎有两种，分别为汽油引擎和电动引擎，继承引擎父类。

这里我们使用 @Inject 注解标记构造函数，告诉 Hilt 如何创建这个类的实例。

当 Hilt 需要提供一个 Engine 实例时，它会调用这个构造函数。在这个例子中，Engine 类没有任何依赖，所以构造函数没有任何参数。

接下来，我们用这些零件拼出来一辆车

```
class Car @Inject constructor(    private val engine: Engine,    private val tire: Tire,    private val horn: Horn,    private val bulb: Bulb)    fun check()}
```
这里我们使用 @Inject 注解标记车的构造函数，一共有四个零件参数待注入，我们会在之后的@provider获取实例方法种将其注入，在注入时会便会调用@inject标记的构造函数

```
object CarStore@Module@InstallIn(SingletonComponent::class)object CarFactory }
```
这里我们定义了一个CarStore，用来存储汽车的类型，使用一个可变列表存储了汽油引擎、电动引擎、汽油引擎、电动引擎。

最后，我们用@Module标记了一个CarFactory作为提供依赖的工厂，里面包含一个用于提供依赖的方法，@InstallIn标记则代表这个模块安装在SingletonComponent::class组件中。

当一个Car变量用@Inject 表示需要注入后，Hilt便会调用Car变量对应的@Provider函数进行注入。这里在makeCar()函数中，每次会从CarStore的列表中获取一个实例引擎，并将它注入到实例化的汽车中，而轮胎、喇叭、车灯则会直接获取实例注入。

最后，我们在主函数中去获取车辆

```
@AndroidEntryPointclass MainActivity : ComponentActivity()                 }            }        }        car1.check()        car1.start()        car2.check()        car2.start()        car3.check()        car3.start()        car4.check()        car4.start()    }}
```
lateinit var car1: Car使用了@Inject标注表示需要注入，则会调用@Provide标注的方法对其进行注入，获取car实例，其余同理

最终系统打印如下：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RSYpfibibsrNsYj9r5nqVjyf25FbhrEAos87oibNSeVesCvlmPk47B1WTs6ca7mnLtwcLR3NMPDsldg/640?wx_fmt=png)看起来十分简单，我们只是将变量的实例化变得简单了一点，但这只是Hilt实例化注入初步的一次小尝试，待到进阶的项目中，你就会发现Hilt的神奇作用

### 参考资料

[1]Java 8 功能: https://developer.android.google.cn/studio/write/java8-support?hl=zh-cn

[2]Application: https://developer.android.google.cn/reference/android/app/Application?hl=zh-cn

