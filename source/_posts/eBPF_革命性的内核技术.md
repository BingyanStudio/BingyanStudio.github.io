---
title: 'eBPF: 革命性的内核技术'
cover_image: /img/cover.png
abbrlink: 79147fd0
date: 2023-10-14 08:43:41
---

## 介绍

eBPF（extened Berkeley Packet Filter）是一种内核技术，它允许开发人员在不修改内核代码的情况下，在运行时向操作系统添加功能。最初 BPF 只是一种网络过滤器，可以捕获和过滤网络数据包。eBPF 在它的基础上进行扩展，能力已远超数据包过滤。

## 用途

* 流量处理：eBPF 可以用于捕获网络数据包，分析网络流量，对网络数据包进行安全过滤，为容器提供高性能的网络等。代表项目有 Cilium，基于 eBPF 为云原生环境提供高性能、安全、可观测的网络，不久前它还正式从 CNCF 毕业。

* 性能分析：eBPF 可以用于收集内核和应用程序的运行指标，并对它们进行分析。代表项目有 Pixie，k8s 上的观测与诊断平台，可自动收集集群上的网络请求、数据库查询等各种指标，还有无需重新编译部署就能调试 Golang 程序的炫酷功能。

* 安全检测：eBPF 可以用于跟踪和收集应用程序的行为，以发现潜在的风险操作，保障数据的安全可靠，代表项目有 Falco、Tracee。

借助 eBPF 还可以实现一些其它的功能，用途非常广泛！

## 原理

eBPF 程序是事件驱动的，通常会挂载到一个内核钩子（hook）上，比如系统调用、函数进入/退出、网络事件等，还可以创建内核探测（kprobe）或用户探测（uprobe）来附加到内核或用户程序的任何地方。

eBPF 程序需要在内核中运行，通常使用一个用户态的程序，通过系统调用，来将编译后得到的字节码复制到内核空间中。

加载到内核空间后，还需要进行验证，以确保它不会进行恶意操作，如系统调用、内存访问等，保证 eBPF 程序能够安全运行。

最后字节码将通过即时编译（JIT）转换为特定机器的指令集，以优化运行速度。

此外，eBPF Maps 允许 eBPF 程序在调用之间保持状态，以进行相关的数据统计，并与用户空间的应用程序共享数据。

## 实践

大部分情况下，我们不会直接编写 eBPF 程序，而是利用 Clilium、bcc 或者 bpftrace 等在 eBPF 之上提供抽象的项目来编写，避免和原始的复杂代码打架。下面我们以 bpftrace 为例来进行实际操作：

1. 准备环境并安装 bpftrace：sudo apt install bpftrace

2. 编写一个极其简单的 golang 程序

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
```
// demo.go package main// 禁用内联//go:noinlinefunc hello(n int) }func main() 
```
1. 编译一下：go build demo.go

2. 使用 bpftrace 查看可跟踪函数列表：

* 
* 
* 
```
$ sudo bpftrace -l 'uprobe:./demo:main*'uprobe:./http_demo:main.hellouprobe:./http_demo:main.main
```
```

```
1. 使用 bpftrace 收集并输出调用参数

* 
* 
* 
* 
* 
```
sudo bpftrace -e 'uprobe:./demo:main.hello'# reg('ax')代表第一个参数，具体可参考 Go internal ABI# 更多命令参考bpftrace文档# 然后运行一下这个程序./demo
```
```

```

可以看到，eBPF 程序输出了 hello 函数被调用时的参数。

## 总结

eBPF 作为一种新兴的内核技术，有着灵活、安全、应用场景广阔等许多优点，在云原生网络、可观测等领域更是有着无可比拟的优势。缺点可能就是学习路线太过陡峭，uprobe 方案通用性较差且存在性能消耗之类的小问题吧。

## 参考

* What is eBPF? An Introduction and Deep Dive into the eBPF Technology

* eBPF 介绍 | 酷 壳 - CoolShell

* zoidbergwill/awesome-ebpf: A curated list of awesome projects related to eBPF. (github.com)

* iovisor/bcc: BCC - Tools for BPF-based Linux IO analysis, networking, monitoring, and more (github.com)

* iovisor/bpftrace: High-level tracing language for Linux eBPF (github.com)

* Tracing golang with bpftrace – Mechpen

