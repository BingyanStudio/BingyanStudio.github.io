---
title: node 包管理工具全面分析
cover_image: /img/cover.png
abbrlink: 8a210ff8
date: 2022-04-15 13:25:06
---

js/node 生态如此繁荣，包管理工具功不可没，最早的包管理工具是 npm（Node Package Manager）。2010 年 npm 正式发布，并被 node 支持，其主要贡献有三：

1. 提供统一的开发者社区
2. 提供网站来托管包
3. 提供 cli 供开发者使用
可以说是功不可没了

但是！但是在前端变得如此复杂且追求极致的环境下，npm 的一些缺点逐一显现，出现了更高效更好用的包管理工具，本文将对 npm / yarn / pnpm 展开客观公正的分析

## npm

主要就是说说 npm 的不足 ;P

### npm 2：缓慢的下载速度，可能存在的文件名过长 bug

说到 npm 就不得不说那个大名鼎鼎的 node_modules 文件夹：

![](https://mmbiz.qpic.cn/mmbiz_jpg/rxLIic6e5g8S2x2UdfGr426icBj6r1iattWCIjFFhOa3HBsjQUgjnexk9To0RSw17CCmCGebQNdYKAibsNnTFq6dtg/640?wx_fmt=jpeg)node_modules-meme![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S2x2UdfGr426icBj6r1iattWVEWV2OZ8PiayFQiciaP9DjRF7ZgakBbZxUicUiajyKW6UfU7ericeUrxPMYw/640?wx_fmt=png)node_modules> 我真没刻意往里面放什么大文件，他真的就是这么大 那么问题来了，node_modules 为啥这么大呢？
> 
> 其实这个问题也可以理解，毕竟依赖是层层嵌套的，你依赖了 react，react 又会递归的依赖别的包，一层层套下去呈树状结构。npm（npm 2 及以前版本） 读到 package.json 中的依赖项后就会去下载依赖，并根据其树结构组织文件夹：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S2x2UdfGr426icBj6r1iattWmSNIW7ZEoMNRcSSiczPbjnnHaibU7Lma8ibuZiavOqScb9l4SZtMdWSdPQ/640?wx_fmt=png)tree-struct可以看到， vue 和 react 依赖的 foo 版本有较大出入，不能复用，但是 bar 的版本只有小版本的不同，npm 却还是把 bar 下载&存储 了两遍，因此带来了存储和下载时间的双重负担不仅于此，嵌套的树结构导致文件路径名称可能变得非常长，以至于超出一些处理程序的上线（主要见于 windows 系统）

### npm 3：长时间的依赖计算，不安全的依赖

npm 3 及以后版本采用了不一样的思路：使用扁平化存储结构

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S2x2UdfGr426icBj6r1iattW5p3ibqKtaic0l4wsgkyQstRPcOjqYYlXprIbYAoRSZ77NFNvTERUAGYg/640?wx_fmt=png)flatten诶，但是这样两个包依赖的 foo 的大版本不同，在扁平化之后却依赖了同一个版本，这样不会有问题吗？确实，这就是 npm 使用扁平化后带来的安全问题，尽管 npm 会根据语义化版本依赖尽力计算出满足尽可能多要求的最合适版本，但这只是尽力而为，不能保证每个依赖的版本都满足要求，也不能保证不同机器上安装的依赖是一样的除了安全问题，这个计算版本的过程本身也是相当耗时的，是 npm 安装速度慢的一个很重要的原因

除了安全问题，扁平化还带来了幽灵包的问题：现在当依赖 import 一个子依赖，他会从 node_modules 文件夹下查找，但是由于扁平化的处理，node_modules 文件夹下还有很多别的包的依赖，这样一来，一个依赖 import 未声明在自己的 package.json 下、却已经被别的依赖安装好的子依赖 是不合逻辑但实际可行的

缓解这个问题的方法是，npm shrinkwrap 指令，该指令会生成 npm-shrinkwrap.json 文件来在不同机器上统一依赖版本，同时简化依赖计算的工作量

### npm 5：仍存在问题

npm 5 将锁定依赖版本的 package-lock.json 设为默认行为（其功能和上文提到的 npm-shrinkwrap.json 相同npm 5 还增加了 security vulnerabilities checks，会检查可能存在版本问题的依赖项并提醒你可能存在的问题（让你知道自己怎么死的 bushi），npm 6 很贴心的增加了 npm audit 命令来主动统计易损的依赖，并有nom audit fix 命令来尝试修复之（听起来也不那么可靠...）

### npm 总结

总结一下就是，即使是现在最新版的 npm 也存在以下问题：

1. 不安全的依赖
2. 下载+分析速度慢
3. node_modules 庞大
## yarn

yarn 是 2016 年推出的新一代包管理工具，其主要改进点在于使用了并行下载（相比而言 npm 是串行依次下载的），提升了下载速度

网上文章说了不少 yarn 的优点，但是其实 npm 也有在吸取其优点，现在这些差距已经被抹平了，不吹不黑的话，现在（2022/2/22）yarn 主要的优势就在于并行下载了

此外，yarn 还有一个神奇的 feature：PnP（Plug and Play）

### PnP

从 node_modules 里查找依赖是整个 node 生态的默认行为，当你引入一个包，node 会一层层往父目录的 mode_modules 文件夹下查找这个包，直到找到它或者找到根目录都没找到而报错...似乎整个生态都默认了 node，一定要有 node_modules

但是 yarn pnp 不这么认为，首先我们认识到，使用 node_modules 文件夹存储依赖会有以下问题：

1. node_modules 太大了！将文件写入其中的 IO 开销巨大
2. 上述查找包的过程也涉及多次文件夹读取和文件查找，同样有很大开销
3. 对下载的依赖去重会很复杂（采用扁平结构和树状结构都有弊端）
yarn pnp 采用了全新的包管理策略：PnP，用一个 .pnp.js 文件代替整个 node_modules 文件夹！

我们试试这个 pnp 模式：

```
yarn init -yyarn add foo --pnp
```
生成了 package.json、.pnp.js，而没有 node_modules！

.pnp.js文件是个几千行大小的 js 文件，其中和依赖包相关部分为：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S2x2UdfGr426icBj6r1iattWxUhwlL1aLsLgPicZLCzqZO9Un4bNcoowb7GtiaUBhMbHNgYmaUKfkymA/640?wx_fmt=png)pnp.js要运行项目必须使用 yarn xxx 命令来运行，yarn 会自动执行.pnp.js文件来处理依赖，而不用开发者下载，开箱即用（即 Plug and Play）

### yarn 总结

1. yarn 的并行下载解决了 npm 下载缓慢的问题
2. 和 npm 一样都有 lock 文件来锁定版本
3. 和 npm 一样有依赖不安全的问题，同时 node_modules 的大小还是很大
4. yarn PnP 解决了第 3 点问题，但是引入的问题在于这个模式太反常规了，存在兼容性问题，一些包在 pnp 模式下出现依赖错误，目前接受度不高
> 小 tip：为了让开发者更平滑的迁移到 yarn，yarn 提供 yarn import 指令来将 package-lock.json 转化成 yarn.lock 文件这何尝不是一种 ntr
> 
> ## pnpm

> performant npm，重头戏来了 对于 npm 的种种弊端，pnpm 采取了神奇的解决思路：将每个项目的所有依赖下载在统一的目录下，而在项目的 node_modules 中放上指向那个统一目录的硬链接（可以近似理解为快捷方式）：
> 
> ![](https://mmbiz.qpic.cn/mmbiz_jpg/rxLIic6e5g8S2x2UdfGr426icBj6r1iattWCXJCSlLEUsvr1ARKwyD5MNqQ8Jml2oWEiblMheVn78bDfnpcepgr6pQ/640?wx_fmt=jpeg)pnpm-store这样带来的好处是：

1. 尽可能的复用包，避免了在多个项目中反复下载&存储同一份包，降低了存储占用
2. install 的时候只是创建硬链接而不是复制文件，大大提高下载速度
3. 使用树状结构而不是扁平结构，带来了依赖的安全性
4. 如何避免过长路径的问题？pnpm 会对一些包进行提升，让他不在那么深的位置
个人使用经验是，我将电脑上 10 余个项目全部迁至 pnpm， c 盘的存储空间增加了 20 多 G，同时 pnpm 下载速度快到飞起，越往后越快（能复用的包越多）

此外，pnpm 还有以下优点 😎：

1. 他有 95% 翻译度的中文官网
2. 官网支持深色模式
3. 官网 UI 还挺好看
4. 官网还有响应式布局
> 读者：好了你憋说了，知道你喜欢 pnpm 了 有人说 pnpm 几个字打起来不顺手？
> 
> ```
# 这何尝不是一种 ntralias npm="pnpm";alias yarn="pnpm";alias whatever_U_like="pnpm";
```
## 总结

| 指标 | npm | yarn | pnpm |
| --- | --- | --- | --- |
| 速度 | base line | 多线程 | 硬链接而非复制文件 |
| 安全性 | 靠 lock 文件，不可靠 | 靠 lock 文件，不可靠 | 安全 |
| 存储 | node_modules 散落在各项目，存储占用大 | node_modules 散落在各项目，存储占用大 | 只有一份 node_modules |

个人感觉是 pnpm > yarn > npm

附上这个库做的各个包管理器下载速度比较：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S2x2UdfGr426icBj6r1iattWjkgABgibYh4Xs5qTBAZENdybicBNYTkvTKVL7m0mZbTSAic408AkrRTSw/640?wx_fmt=png)benchmark可见 pnpm 速度雀食快！

## 还等什么，npm i -g pnpm 呀！

## 参考

* Yarn vs npm: Everything You Need to Know
* State of Yarn 2 (Berry) in 2021
* 为什么现在我更推荐 pnpm 而不是 npm/yarn?
* 一文看懂 npm、yarn、pnpm 之间的区别

