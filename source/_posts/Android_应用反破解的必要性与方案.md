---
title: Android 应用反破解的必要性与方案
cover_image: /img/cover.png
abbrlink: bf42877d
date: 2023-04-02 09:30:10
---

# 

# 反破解的必要性

## 内购应用/提供会员机制的应用

应用被破解直接造成付费用户的不平衡与经济损失

## 内嵌广告应用

被破解去广告直接导致经济损失

## 无广告、无收费

无基础的安全防护易被他人恶意植入广告SDK后上架到应用商店赚取广告费(亲身经历)

# 反破解常规方案

## 一、混淆

使用各种字典替换掉有意义的名称等，使破解者难以找到想要的内容

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8T6jOWVQFpRPofvO0G7Fr5jEI8dAcTJEP26lQpZ5s2iazSIqic1RRoLibHkuXZian6cqEPicicFENXXMyoQ/640?wx_fmt=png)未混淆的内容![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8T6jOWVQFpRPofvO0G7Fr5jZibLzHRiafaSc26qoW3FAApg8Eh6qIuwdFWKnwmtXk7G9icnQvOclicU2w/640?wx_fmt=png)混淆词典![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8T6jOWVQFpRPofvO0G7Fr5jd11CDIUjFmjD81hcRh4icLXbDmSX4mGoNyCtLE2EW5TB67oWiaWiaer7Q/640?wx_fmt=png)混淆后的包![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8T6jOWVQFpRPofvO0G7Fr5jITLzialS8ppFIMWn0OJ6Hm7oPFj6g9wXzNTib89FG5nbWPfJNkyHaOOw/640?wx_fmt=png)混淆后的类和方法![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8T6jOWVQFpRPofvO0G7Fr5jdnZFXQthUx8U3iamLiaTuUiaaB1GK4Q6FTcGrPWia09uiaM8UdThMMtzMLw/640?wx_fmt=png)一些奇怪的混淆词> 使用一些奇怪的混淆词作为字典可以令反编译者沉浸其中，无心分析代码
> 
> ## 二、签名校验

Android 的 apk 安装包本质上是一个zip压缩包，但需要签名才能够进行安装，签名一般就是使用特定的签名文件计算apk内各个文件的散列值(一般使用SHA-1摘要算法)，签名文件类似于密钥，破解者一般无法获得(社工方法除外)，apk内的文件一旦发生改变签名便无法校验通过进而无法安装，因此破解者需要对安装包重新使用自己的签名文件进行签名。

App内可以通过代码获取到签名文件的散列值，一旦获取到的签名文件散列值与预期中的不一致就可以说明安装包已经被修改了。

签名校验主要是针对于二次打包的检测防范措施，如果没有签名校验和完整性校验功能，应用可能被恶意攻击者二次打包，被盗版的风险大大增加，同时也可能进行任意代码修改。

### 优点

1. 可以有效杜绝被一些通用工具一键植入广告
2. 简单易行
### 缺点

1. Java层获取签名的方法比较固定，配合Java的反射机制和Hook机制，破解者会直接Hook伪造代码中获取到的签名(相当于外挂一段代码直接拦截掉获取签名的方法)，使验证代码失效。
2. 原生层获取签名的成本较高，一般也很少采用。
### 解决方案

Android如何把签名校验做到极致https://blog.csdn.net/qq_40948137/article/details/115866451

## 三、Dex校验

Android 程序中的所有代码都会编译至dex文件中，想要破解App就必然会修改dex文件，dex的散列值就会发生变化，我们可以在打包完成后将dex的散列值写入一个文件然后添加到安装包中，之后在代码中读取dex文件并进行校验其散列值即可。

为了提高安全性和操作便利性，也可以将校验值存入云服务器，客户端本地将Dex散列码发送给服务器，由服务器进行校验，不过需要App本身主要功能联网，不然直接禁用网络权限将是最快的破解方法。

### 优点

方法不常用，破解者难以想到

### 缺点

每次打包完成以后需要向包中写入散列值文件，写入以后因为文件变动还需要重新进行签名，比较麻烦。或将Dex散列码存储在服务器中。可以通过一些自动化流程来简化步骤。

## 四、加壳

壳本质上就是一个加载器。

对原本的dex文件进行加密，替换原dex文件为提供商写好的解密dex文件，运行时解密dex对加密后的原dex文件进行解密，之后再运行解密后的dex文件

* 未加壳前：系统直接执行原dex，即原apk
* 加壳后：系统执行 壳代码 --> 脱壳得到原dex --> 执行原dex
### 优点

* 对安卓应用加壳可以有效避免软件被破解、动态注入、调试或被篡改等。
### 缺点

1. 影响运行效率
2. 因为运行时还是需要解壳，市面上免费的加壳解决方案极其容易被一些一键脱壳工具在运行时读取内存中的 dex 文件进行脱壳
## 五、使用新语言/新框架

反编译工具一般都支持把 smali 字节码反编译回Java代码，考虑使用 Kotlin 进行开发，Kotlin 编译出的 smali 代码回编译回 Java 后会相应增加一些阅读难度，特别是 Kotlin 的协程部分反编译回 Java 后会变成回调地狱，较复杂的计算逻辑写在协程中既可以优化性能，又能给破解者造成麻烦，一举两得。

另外，Kotlin 支持静态实例与顶层函数，可以把一些校验代码分别写在不同的静态实例内，配合静态实例的自动初始化和混淆，可以做到把检验代码藏到难以找到的地方。同时配合 Kotlin，进一步加大了阅读查找难度。

除此以外，现阶段的App都是基于传统的XML布局实现的，破解者可以通过 Activity (俗称界面/页面)快速定位到相关代码。可以考虑通过 Jetpack Compose 进行界面UI开发，Jetpack Compose 完全抛弃了传统的XML布局方案，所有界面均以 Kotlin 代码的形式存在（且 Compose 界面代码编译为 smali 代码后反编译回 Java 的代码基本无法阅读，配合混淆后基本无法定位 UI 界面文件），只需要一个 Activity，配合导航框架，即可完成整个App，破解者难以找到想要的界面。

另外 Kotlin 与 Jetpack Compose 同时也能够大幅提高工作效率，同时也是趋势。不存在为了反破解使用一些难用的工具的问题。

同时，一些跨端框架如 Flutter / Xmarin 等也能一定程度上起到保护代码的目的。

