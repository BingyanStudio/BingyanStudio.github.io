---
title: ​用 Rust 写一个操作系统
cover_image: /img/cover.png
abbrlink: 5dc710b2
date: 2023-12-25 12:00:21
---

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RxzqpdblHDF7icpealUly2goNICMXTyYrykRW4N7W7gODyX4yIcXddxzrASYQwJibNtPboJFNWmG2Q/640?wx_fmt=png&from=appmsg)如何自己做一个操作系统？笔者自己刚跟着 Philipp Oppermann[1] 搓完半个小操作系统，这里作出一些用 Rust 写操作系统的总结，感兴趣的同学可以移步原文了解更多细节。## 禁用标准库

操作系统是直接与硬件接触的程序，这意味着我们不能直接使用语言提供的标准库，因为标准库中依赖了宿主系统的系统能力，如内存分配、任务调度等。为了用 Rust 编写一个操作系统内核，我们需要创建一个独立于操作系统的可执行程序。这样的可执行程序常被称作独立式可执行程序（freestanding executable）或裸机程序(bare-metal executable)。Rust 编译器默认自动引入标准库，可以用 `no_std` 属性禁用标准库。```
// main.rs#![no_std]fn main() {}
```
编译器仍然报错了，因为在禁用标准库后编译器缺少一个 `#[panic_handler]` 函数和一个 `language item`，这在标准库中是被预先实现的。```
use core::panic::PanicInfo;/// 这个函数将在 panic 时被调用#[panic_handler]fn panic(_info: &PanicInfo) -> ! }
```
实现 `panic_handler` 时注意函数返回类型为 `!`，即 Rust 中的 `Never Type` 这意味着函数永远不会返回，这是因为 panic 后程序已经无法继续执行了。### Language Item 语言项

语言项是一些编译器需求的特殊函数或类型。例如 Rust 的 Copy trait 是一个语言项，告诉编译器哪些类型需要遵循复制语义（copy semantics），查看 Copy trait 的实现，我们会发现一个特殊的 #[lang = "copy"] 属性将它定义为了一个语言项，达到与编译器联系的目的。#### eh_personality 语言项

该语言项与栈展开（stack unwinding）相关，栈展开是一种异常处理机制，当程序发生 panic 时，栈展开会将栈上的局部变量逐个销毁，直到回到 panic 之前的状态。这个过程需要编译器生成一些额外的代码，这些代码需要依赖 eh_personality 语言项。但这也依赖操作系统的能力，编写我们自己的系统时暂时关闭栈展开。要禁用栈展开需在 Cargo.toml 中添加如下配置```
[profile.dev]panic = "abort"[profile.release]panic = "abort"
```
#### start 语言项

在一个典型的使用标准库的 Rust 程序中，程序运行是从一个名为 crt0 的运行时库开始的。crt0 意为 C runtime zero，它能建立一个适合运行 C 语言程序的环境，这包含了栈的创建和可执行程序参数的传入。之后，这个运行时库会调用 Rust 的运行时入口点，这个入口点被称作 start 语言项。我们的独立式可执行程序并不能访问 Rust 运行时或 crt0 库，所以我们需要定义自己的入口点。只实现一个 start 语言项并不能帮助我们，因为这之后程序依然要求 crt0 库。所以，我们要做的是，直接重写整个 crt0 库和它定义的入口点。要告诉 Rust 编译器我们不使用预定义的入口点，我们可以添加 #![no_main] 属性。同时，我们需要定义一个 _start 函数作为程序的入口点。`_start` 这个名字是一个默认的约定，它将被引导程序调用启动我们的系统。```
#[no_mangle]pub extern "C" fn _start() -> ! }
```
`#[no_mangle]` 告诉编译器禁用名称重整（以 C 风格保持函数名称）## 延迟初始化

在编写系统代码过程中经常需要创建一些全局静态变量，比如 GDT 全局描述符表，VGA buffer 等，但 Rust 中的全局变量一般是不可变的，对可变静态变量的操作都会被视为 `unsafe` 因为这可能导致数据竞争。但有时我们需要在程序启动后才初始化这些变量，这时就需要用到 `lazy_static` crate。`lazy_static` crate 主要提供了 `lazy_static!` 宏，它允许我们在程序启动后初始化一次全局变量，同时保持变量的不可变性。例如：```
lazy_static!         idt    };}
```
## spin::Mutex

上述提到的 `lazy_static` 解决了 static 变量初始化的问题，但是当我们确实需要在程序运行时修改全局变量时，就需要用到 `Mutex` 了。一般的 Mutex 实现中，它通过提供当资源被占用时将线程阻塞（block）的互斥条件（mutual exclusion）实现这一点，但我们自己的系统尚未实现线程的概念，所以这里使用的是 `spin::Mutex`，它通过自旋（spin）的方式实现互斥条件，即当资源被占用时，线程不会被阻塞，而是一直循环尝试获取资源，直到获取到资源为止。```
use spin::Mutex;lazy_static! ,    });}// When to use MutexWRITER.lock() // : MutexGuard 是可变类型    .do_something();
```
Rust 高性能、内存安全、并发安全的特性使得它成为一个非常适合编写操作系统的语言，本文只是略述了在写操作系统时用到的高级特性，使得在保持安全性的同时提供了对底层的强大控制力，更多细节可以参考 https://os.phil-opp.com/[2] 。### 参考资料

[1]Philipp Oppermann: https://os.phil-opp.com/

[2]https://os.phil-opp.com/: https://os.phil-opp.com/

