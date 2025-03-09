---
title: Worker.js
cover_image: /img/cover.png
abbrlink: 141cecf6
date: 2023-05-17 04:06:39
---

# Worker.js

Web Workers 可以让代码独立于主线程运行，避免大量的运算阻塞浏览器渲染画面，并且可以在计算过程中交互。其实它的功能和API非常简单，就是多线程工具。

## 如何使用

### 网页端

1. 创建 Worker
```
let worker = Worker("/path/to/worker.js")
```
1. 接收来自 Worker 的信息
```
worker.onmessage = event => 
```
1. 向 Worker 发送信息
```
worker.postMessage("Hello!")
```
### worker.js

1. 接受信息
```
onmessage = event => }
```
1. 发送信息给网页
```
postMessage("How are you?")
```
### 结合起来...

```
// 
```

