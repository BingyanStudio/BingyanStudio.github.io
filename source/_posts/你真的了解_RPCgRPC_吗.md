---
title: 你真的了解 RPC/gRPC 吗？
cover_image: /img/cover.png
abbrlink: c03c04c6
date: 2023-12-08 12:12:32
---

镇楼图

## 引

远古时期，每个进程各干各的，但随着发展有时候会存在 A 进程调用 B 进程某一方法，使用其功能的场景，但是最初是不支持的，就产生了所谓的 IPC（Inter-process communication 本地进程间通信），诸如管道、信号量、共享内存、套接字等。

再后来越来越多的单机系统复杂到无法维护面临拆分，小型机的瓶颈凸显及性价比越来越低，由 PC 和廉价服务器构成的集群、分布式方案逐渐形成，开始出现多个 PC 或者服务器搭建分布式系统的场景，之前单机上的 IPC 也演变成了现在的 RPC。

## 什么是 RPC？

RPC（Remote Procedure Call）远程过程调用协议是一个用于建立适当框架的协议。从本质上讲，它使一台机器上的程序能够调用另一台机器上的子程序，而不会意识到它是远程的。

RPC 是一种软件通信协议，一个程序可以用来向位于网络上另一台计算机的程序请求服务，而不必了解网络的细节。RPC 被用来像本地系统一样调用远程系统上的其他进程。过程调用有时也被称为函数调用或子程序调用。

## RPC 如何工作？

在远程调用时，我们需要执行的函数体是在远程的机器上的，这会给调用这个函数带来一些问题：

1. 首先客户端需要告诉服务器，需要调用哪个函数，这里函数和进程 ID（Call ID）存在一个映射，客户端远程调用时，需要查一下函数，找到对应的 Call ID，然后执行函数的代码。

2. 客户端需要把本地参数传给远程函数，本地调用的过程中，直接压栈即可，但是在远程调用过程中不在同一个内存里，无法直接传递函数的参数，因此需要客户端把参数转换成字节流，传给服务端，然后服务端将字节流转换成自身能读取的格式，是一个序列化和反序列化的过程。

3. 数据准备好了之后，如何进行传输？网络传输层需要把 Call ID 和序列化后的参数传给服务端，然后把计算好的结果序列化传给客户端，可以使用 TCP 协议完成，gRPC 使用的是 HTTP/2 协议。

具体过程如下：

Client 端：

```
int result = Call(ServerAddr, func, param1, param2)
```
1. 将这个调用映射为 Call ID

2. 将 Call ID, param1, param2 序列化，可以直接将它们的值以二进制形式打包

3. 把 2 中得到的数据包发送给 ServerAddr，这需要使用网络传输层

4. 等待服务器返回结果

5. 如果服务器调用成功，那么就将结果反序列化，并赋给 result

Server 端：

1. 在本地维护一个 Call ID 到函数指针的映射 call_id_map，可以用 std::map>

2. 等待请求

3. 得到一个请求后，将其数据包反序列化，得到 Call ID

4. 通过在 call_id_map 中查找，得到相应的函数指针

5. 将 param1, param2 反序列化后，在本地调用 func 函数，得到结果

6. 将结果序列化后通过网络返回给 Client

Learn more: RPC原理与Go RPC https://www.liwenzhou.com/posts/Go/rpc/

## 常见的 RPC 框架

一类是跟某种特定语言平台绑定的，另一类是与语言无关即跨语言平台的。

跟语言平台绑定的开源 RPC 框架主要有下面几种。

* Dubbo：国内最早开源的 RPC 框架，由阿里巴巴公司开发并于 2011 年末对外开源，仅支持 Java 语言，广泛用于解决分布式服务治理问题。

* Motan：微博内部使用的 RPC 框架，于 2016 年对外开源，仅支持 Java 语言，致力于简化分布式系统之间的远程调用的RPC框架。

* Tars：腾讯内部使用的 RPC 框架，于 2017 年对外开源，仅支持 C++ 语言。

* Spring Cloud：国外 Pivotal 公司 2014 年对外开源的 RPC 框架，仅支持 Java 语言

而跨语言平台的开源 RPC 框架主要有以下几种。

* gRPC：Google 于 2015 年对外开源的高性能、跨语言 RPC 框架，支持多种语言。

* Thrift：最初是由 Facebook 开发的内部系统跨语言的 RPC 框架，2007 年贡献给了 Apache 基金，成为 Apache 开源项目之一，支持多种语言。

* hprose：一个MIT开源许可的新型轻量级跨语言跨平台的面向对象的高性能远程动态通讯中间件。它支持众多语言:nodeJs, C++, .NET, Java, Delphi, Objective-C, ActionScript, JavaScript, ASP, PHP, Python, Ruby, Perl, Golang 。

Learn more: RPC框架对比 https://juejin.cn/post/7107070593383530533

## gRPC 又是何方神圣？

gRPC 一开始由 google 开发，是一款语言中立、平台中立、开源的远程过程调用(RPC)系统。

使用 gRPC， 我们可以一次性地在一个 .proto 文件中定义服务并使用任何支持它的语言去实现客户端和服务端，反过来，它们可以应用在各种场景中，从 Google 的服务器到你自己的平板电脑—— gRPC 帮你解决了不同语言及环境间通信的复杂性。使用 protocol buffers 还能获得其他好处，包括高效的序列化，简单的 IDL 以及容易进行接口更新。总之，使用 gRPC 能让我们更容易编写跨语言的分布式代码。

## gRPC 如何工作？

### Protocol Buffers

> 接口描述语言（Interface description language，缩写IDL），是用来描述软件组件接口的一种计算机语言。IDL通过一种独立于编程语言的方式来描述接口，使得在不同平台上运行的对象和用不同语言编写的程序可以相互通信交流；比如，一个组件用C++写成，另一个组件用Java写成。
> 
> protobuf 协议是跨语言跨平台的序列化协议。我们经常使用的 json、xml 等都是一种序列化的方式，只是他们不需要提前预定义 IDL（规定数据的结构和格式），且具备可读性，当然他们传输的体积也因此较大，可以说是各有优劣。

protobuf 一个简单的例子：

```
syntax = "proto3";message SearchRequest 
```
定义消息类型：

* 类型：类型不仅可以是标量类型（int、string 等），也可以是复合类型（enum 等），也可以是其他 message

* 字段名：字段名比较推荐的是使用下划线/分隔名称

* 字段编号：一个 message 内每一个字段编号都必须唯一的，在编码后其实传递的是这个编号而不是字段名

* 字段规则：消息字段可以是以下字段之一

* singular：格式正确的消息可以有零个或一个字段（但不能超过一个）。使用 proto3 语法时，如果未为给定字段指定其他字段规则，则这是默认字段规则

* optional：与 singular 相同，不过您可以检查该值是否明确设置

* repeated：在格式正确的消息中，此字段类型可以重复零次或多次。系统会保留重复值的顺序

* map：这是一个成对的键值对字段

* 保留字段：为了避免再次使用到已移除的字段可以设定保留字段。如果任何未来用户尝试使用这些字段标识符，编译器就会报错

### HTTP/2.0

测试：http://www.http2demo.io/，打开 Dev Tools 查看 Network 一栏可以很明显发现 HTTP/1 和 HTTP/2 的区别。

相比于 HTTP/1.1，HTTP/2.0 有哪些优势？

1. 二进制分帧

HTTP/2 采用二进制格式传输数据，而非 HTTP/1.x 的文本格式，二进制协议解析起来更高效。HTTP/2 中，同域名下所有通信都在单个连接上完成，该连接可以承载任意数量的双向数据流。每个数据流都以消息的形式发送，而消息又由一个或多个帧组成。多个帧之间可以乱序发送，根据帧首部的流标识可以重新组装。

1. 多路复用

多路复用代替原来的序列和阻塞机制。所有就是请求的都是通过一个 TCP 连接并发完成。HTTP 1.x 中，如果想并发多个请求，必须使用多个 TCP 链接，且浏览器为了控制资源，还会对单个域名有 6-8个的TCP链接请求限 制。

1. 服务器推送

服务端可以在发送页面HTML时主动推送其它资源，而不用等到浏览器解析到相应位置，发起请求再响应。

2. 头部压缩

HTTP/1.1 请求的大小变得越来越大，有时甚至会大于 TCP 窗口的初始大小，因为它们需要等待带着 ACK 的响应回来以后才能继续被发送。HTTP/2 对消息头采用 HPACK（专为 HTTP/2 头部设计的压缩格式）进行压缩传输，能够节省消息头占用的网络的流量。而 HTTP/1.x 每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。

## gRPC 试吃！

首先，通过 protocol buffers 定义服务：

```
syntax = "proto3";option java_package = "io.grpc.examples";package helloworld;// The greeter service definition.service Greeter }// The request message containing the user's name.message HelloRequest // The response message containing the greetingsmessage HelloReply 
```
运行以下命令生成 RPC 代码：

```
$ protoc --go_out=. --go_opt=paths=source_relative \--go-grpc_out=. --go-grpc_opt=paths=source_relative \pb/hello.proto
```
项目目录结构

```
hello_server├── go.mod├── go.sum├── main.go└── pb    ├── hello.pb.go    ├── hello.proto    └── hello_grpc.pb.go
```
生成后的东西长这样：https://github.com/grpc/grpc-go/blob/master/examples/helloworld/helloworld/helloworld_grpc.pb.go

然后，我们只需实现 SayHello 接口即可

```
type server struct func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloResponse, error) , nil}
```
启动服务：

```
func main() 	s := grpc.NewServer()                  // 创建gRPC服务器	pb.RegisterGreeterServer(s, &server{}) // 在gRPC服务端注册服务	// 启动服务	err = s.Serve(lis)	if err != nil }
```
Client 端操作：

```
package mainimport (	"context"	"flag"	"log"	"time"	"hello_client/pb"	"google.golang.org/grpc"	"google.golang.org/grpc/credentials/insecure")// hello_clientconst (	defaultName = "world")var (	addr = flag.String("addr", "127.0.0.1:8972", "the address to connect to")	name = flag.String("name", defaultName, "Name to greet"))func main() 	defer conn.Close()	c := pb.NewGreeterClient(conn)	// 执行RPC调用并打印收到的响应数据	ctx, cancel := context.WithTimeout(context.Background(), time.Second)	defer cancel()	r, err := c.SayHello(ctx, &pb.HelloRequest)	if err != nil 	log.Printf("Greeting: %s", r.GetReply())}
```
Learn more:

* gRPC 官方文档中文版V1.0 https://doc.oschina.net/grpc?t=58008

* gRPC教程 https://www.liwenzhou.com/posts/Go/gRPC

## 参考

* 到底什么是RPC - 概述 https://cloud.tencent.com/developer/article/1619589

* 千字带你了解什么是 RPC 协议 https://bbs.huaweicloud.com/blogs/337073

* 谁能用通俗的语言解释一下什么是 RPC 框架？- 洪春涛的回答 - 知乎 https://www.zhihu.com/question/25536695/answer/221638079

* RPC框架对比 https://juejin.cn/post/7107070593383530533

* 分布式 RPC 框架比较：dubbo、dubbox、motan、thrift、grpc https://apifox.com/apiskills/comparison-of-rpc-frameworks/

* gRPC 官方文档中文版V1.0 https://doc.oschina.net/grpc?t=58008

* gRPC教程 https://www.liwenzhou.com/posts/Go/gRPC

* 接口描述语言 

* 写给go开发者的gRPC教程-protobuf基础 https://juejin.cn/post/7191008929986379836

* HTTP/2 相比 1.0 有哪些重大改进？- leozhang2018的回答 - 知乎 https://www.zhihu.com/question/34074946/answer/75364178

* 为什么使用HTTP2？https://www.cnblogs.com/jesse131/p/11529931.html

---

