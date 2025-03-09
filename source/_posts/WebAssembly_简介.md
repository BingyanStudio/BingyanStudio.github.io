---
title: WebAssembly 简介
cover_image: /img/cover.png
abbrlink: b68e0da
date: 2023-11-24 12:50:33
---

> 大家好啊，今天我来介绍一种很新的技术：WebAssembly（wasm），并简单尝试使用 Golang 写点 wasm 有意思的小东西。
> 
> ## 

## 

## Wasm 是什么

WebAssembly（缩写为 Wasm）是一种基于堆栈的虚拟机的二进制指令格式。Wasm 被设计为编程语言的可移植编译目标，可以在浏览器或其他提供了 wasm 支持的环境中运行。

* 基于栈的虚拟机中：计算和操作的过程主要依赖于栈数据结构，指令通常包括将数据压入栈、从栈中弹出数据并执行操作，然后将结果再次压入栈中的操作。这种模型简单而高效，适用于许多不同类型的计算，多用于跨平台、安全受限等场景。我们熟悉的 JVM 也是这种。另一种常见的虚拟机是基于寄存器的。

* 二进制指令格式：二进制是固定指令集的表示，无需做词法分析和语法分析，就能直接在 wasm 虚拟机上解释执行。

* 可移植性：代码无需修改即可在不同的平台和环境中运行，wasm 程序可以在任何支持其标准的环境中运行，而无需对其进行修改或重新编译。最初 wasm 设计是在 Web 浏览器上执行，但现在我们有了 WASI（WebAssembly System Interface），一个模块化的系统接口来在 Web 之外运行 wasm，例如访问文件、网络链接等能力。

## 有什么用呢

Wasm 目前已经在浏览器端的图像处理、音视频处理、游戏、IDE、可视化、科学计算等，以及非浏览器端的Serverless、区块链、IoT 等领域有一定的应用，一些现实中的应用例子有：

* 视频播放：爱奇艺直播WebAssembly优化之路 (qq.com)

* 网页应用：Figma is powered by WebAssembly | Figma Blog

* 游戏：Play | WASM-4 (wasm4.org)

* 容器：Wasm workloads (Beta) | Docker Docs

* Serverless：WebAssembly serverless functions in AWS Lambda | Cloud Native Computing Foundation (cncf.io)

* 区块链：WebAssembly & The Future of Blockchain Computing | by Raul Jordan | Medium

* 云原生：基于 WebAssembly 实现 Envoy 与 Istio 的功能扩展 | cloudnative.to

* ......

甚至，还能和之前说的 ebpf 结合，太神奇啦：eunomia-bpf/wasm-bpf: WebAssembly library, toolchain and runtime for eBPF programs (github.com)## 

## 运行时架构

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S48GpJrBu3D0icydcQLOVNe9t0rznPtfG3yw6UJRa6HmfAsKSYGUIhDicrT89pp8b2IFYWFM0cGe5g/640?wx_fmt=png&from=appmsg)* 模块加载和解析器：负责将 WebAssembly 二进制文件加载到内存中，按照规范定义的属性文法进行解码，验证和实例化，生成 WebAssembly 模块对象和运行时数据区域。
* 执行引擎：负责执行 WebAssembly 模块中的字节码指令，管理运行时状态，包括栈解释器，线性内存管理和垃圾回收等功能。
* WebAssembly 接口：负责定义一组可移植、模块化、独立于运行时的 WebAssembly API，使得 WebAssembly 代码可以与外界交互。
### 

### 垃圾回收

对 WebAssembly 而言，初期的主要设计目标是提供一个底层的高效二进制格式及其对应的运行环境，并将静态强类型语言直接静态编译到字节码，避免在语言层面的额外开销，从而提升性能。因此，WebAssembly 没有垃圾收集器，它只提供了一块可以按字节寻址的线性内存。对于基础数据类型，wasm 可以在内存中高效地访问和传递，而对于结构体、数组等复杂的数据结构，需要手动负责对象创建和回收对象或者采用优化的内存分配器来完成内存的管理工作。目前，对这方面的讨论还在激烈进行中，不支持 GC 的人认为没必要，而支持 GC 的人认为：* 采用垃圾收集器，WebAssembly 可以以更快的执行性能，更小的体积支持更广泛的现代高级语言。
* 解放了开发人员对内存管理，提高了内存的安全性。
* WebAssembly 很多场景本质上是一个异构的多语言环境，统一的垃圾收集器可以实现多种语言之间无缝互操作。
而在 2023 年 10 月末，Chrome 已经将使用 GC 设置为默认选项：WebAssembly Garbage Collection (WasmGC) now enabled by default in Chrome - Chrome for Developers。他们的文章里还提到使用 GC 可以优化大小，虽然 C/C++、Rust 不需要 GC，但仍然需要许多代码来管理内存，使得编译产物的体积更大。## 来点例子

在 Golang 中写 wasm，首先我们需要一个编译的工具：TinyGo（事实上 Go 自己也能直接编译 wasm，但是目前的支持似乎还不太好），其次，要在本地运行 wasm，我们还需要一个运行环境：Wasmer: Run, Publish & Deploy any code, anywhere安装完成后就可以开始试试下面的代码啦！### Hello world

```
* 
* 
* 
* 
* 
* 
// main.gopackage mainfunc main() 
```
编译：```
* 
tinygo build -o main.wasm -target=wasm main.go
```
运行：* 
```
wasmer run main.wasm
```
```

```
![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S48GpJrBu3D0icydcQLOVNebM66eLBs3rzIe9PGHfmiceH2V6Db3gdVndmsNOmKaVmY3fctf0ud3vA/640?wx_fmt=png&from=appmsg)### 

### 不写 js 也能写前端

还是上面这段代码，我们可以通过一些方法让它在浏览器里运行：创建一个 index.html：```
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
          Go wasm              
```
把需要的文件复制过来```
* 
cp "$(tinygo env TINYGOROOT)/targets/wasm_exec.js" ./wasm_exec.js
```
在浏览器里打开，就能看到程序输出的结果啦![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8S48GpJrBu3D0icydcQLOVNeHG75W7Qibq2q7iaglxLQtZRyia2ialpHK7ohMVrVvZ1mmwibd0KAxppqhicw/640?wx_fmt=png&from=appmsg)另外 Go 内置的 syscall/js 提供了一些方法来调用 js 函数，我们可以用它实现一些操作，比如：```
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
package mainimport (    "syscall/js"    "time")func GenshinImpact()     js.Global().Get("location").Call("replace", "https://ys.mihoyo.com/")}func main() }
```
![](https://mmbiz.qpic.cn/mmbiz_gif/rxLIic6e5g8S48GpJrBu3D0icydcQLOVNeKMnB5qvNEbSEibaxYZb6pPQ9jM3wPwEt0SFbtUEHiatLoR07PndemJTw/640?wx_fmt=gif&from=appmsg)另外还有把不写 js 贯彻到底的项目：Vugu: A modern UI library for Go+WebAssembly，虽然看起来已经很久没更新了...### 插件系统

参考 wasmerio/wasmer-go: WebAssembly runtime for Go 里的例子：```
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
* 
package mainimport (    "fmt"    wasmer "github.com/wasmerio/wasmer-go/wasmer"    "os")func main() 
```
这下我们可以动态加载各种 wasm 模块了，而且插件不限制编写语言，可以在各种平台上运行，比 Go 自带的 plugin 好太多啦！## 

## 总结

总的来说，WebAssembly 作为一种新兴的技术，具有广泛的应用前景，并且在不断演进和完善中，期待它未来的发展！## 参考文章

[#走进 WebAssembly 的世界 (qq.com)](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzg2ODQ1OTExOA==&action=getalbum&album_id=2847372805888589825&scene=173&from_msgid=2247503361&from_itemidx=1&count=3&nolastread=1#wechat_redirect)![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8TQYPhYwz4Mia5MTNaSIMMjiavB7IcbGAiafKYAhdUicrN6rQuibncGVPicIiaAYWTH3iaZicT3ApAia5VFq8SA/640?wx_fmt=png)

