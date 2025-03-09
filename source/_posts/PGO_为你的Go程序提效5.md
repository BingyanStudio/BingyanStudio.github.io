---
title: 'PGO: 为你的Go程序提效5%'
cover_image: /img/cover.png
abbrlink: 5624c11c
date: 2023-10-27 03:45:40
---

原文：https://go.dev/blog/pgo[1]

2023 年初，Go 1.20 发布了配置文件引导优化（PGO）预览版供用户测试。在解决了预览版中的已知限制，并通过社区反馈和贡献进行了额外的改进后，Go 1.21 中的 PGO 支持已准备好用于一般生产用途！请参阅配置文件引导优化用户指南以获取完整文档。

下面我们将通过一个使用 PGO 来提高应用程序性能的示例。在此之前，我们先来了解一下“配置文件引导优化”到底是什么？

当您构建 Go 二进制文件时，Go 编译器会执行优化，尝试生成性能最佳的二进制文件。例如，常量传播可以在编译时评估常量表达式，从而避免运行时评估成本。逃逸分析避免了本地范围对象的堆分配，从而避免了 GC 开销。内联将简单函数的主体复制到调用者中，通常可以在调用者中进行进一步优化（例如额外的常量传播或更好的逃逸分析）。去虚拟化将对类型可以静态确定的接口值的间接调用转换为对具体方法的直接调用（这通常可以实现调用的内联）。

Go 在各个版本之间不断改进优化，但这样做并不是一件容易的事。有些优化是可调的，但编译器不能在每次优化时都“将其调至 11”，因为过于激进的优化实际上会损害性能或导致构建时间过长。其他优化要求编译器对函数中的“常见”和“不常见”路径做出判断。编译器必须基于静态启发式做出最佳猜测，因为它无法知道哪些情况在运行时会常见。

或者可以吗？

由于没有关于如何在生产环境中使用代码的明确信息，编译器只能对包的源代码进行操作。但我们确实有一个评估生产行为的工具：profile。如果我们向编译器提供配置文件，它可以做出更明智的决策：更积极地优化最常用的函数，或更准确地选择常见情况。

使用应用程序行为配置文件进行编译器优化称为配置文件引导优化 (PGO)（也称为反馈定向优化 (FDO)）。

# 示例

让我们构建一个将 Markdown 转换为 HTML 的服务：用户将 Markdown 源上传到/render，它返回 HTML 转换。我们可以使用`gitlab.com/golang-commonmark/markdown`[2]它来轻松实现这一点。

```
$ go mod init example.com/markdown$ go get gitlab.com/golang-commonmark/markdown@bf3e522c626a
```
创建 main.go 文件：

```
package mainimport (    "bytes"    "io"    "log"    "net/http"    _ "net/http/pprof"    "gitlab.com/golang-commonmark/markdown")func render(w http.ResponseWriter， r *http.Request)     src， err := io.ReadAll(r.Body)    if err != nil     md := markdown.New(        markdown.XHTMLOutput(true)，        markdown.Typographer(true)，        markdown.Linkify(true)，        markdown.Tables(true)，    )    var buf bytes.Buffer    if err := md.Render(&buf， src); err != nil     if _， err := io.Copy(w， &buf); err != nil }func main() 
```
构建并运行这个服务：

```
$ go build -o markdown.nopgo$ ./markdown.nopgo2023/08/23 03:55:51 Serving on port 8080...
```
让我们尝试从另一个终端发送一些 Markdown。我们可以使用 Go 项目中的 README.md 作为示例文档：

```
$ curl -o README.md -L "https://raw.githubusercontent.com/golang/go/c16c2c49e2fa98ae551fc6335215fadd62d33542/README.md"$ curl --data-binary @README.md http://localhost:8080/render# The Go Programming Language

Go is an open source programming language that makes it easy to build simple，reliable， and efficient software.

...
```
## 获取 profile

现在我们已经有了一个可以运行的服务，让我们收集一个配置文件并使用 PGO 进行重建，看看是否可以获得更好的性能。

在 中main.go，我们导入了net/http/pprof[3]，它会自动/debug/pprof/profile向服务器添加一个端点以获取 CPU 配置文件。

通常，您希望从生产环境中收集配置文件，以便编译器获得生产中行为的代表性视图。由于此示例没有“生产”环境，因此我创建了一个简单的程序[4]来在收集配置文件时生成负载。获取并启动负载生成器（确保服务器仍在运行！）：

```
$ go run github.com/prattmic/markdown-pgo/load@latest
```
在负载生成器运行时，从服务器下载一个性能分析:

```
$ curl -o cpu.pprof "http://localhost:8080/debug/pprof/profile?seconds=30"
```
一旦 profile 下载完成，终止负载生成器和服务器。

## 使用 profile

当 Go 工具链找到default.pgo主包目录中指定的配置文件时，它将自动启用 PGO。或者，该-pgo标志go build采用用于 PGO 的配置文件的路径。

我们建议将default.pgo文件提交到您的存储库。将配置文件与源代码一起存储可确保用户只需获取存储库（通过版本控制系统或通过go get）即可自动访问配置文件，并且构建保持可重现。

让我们构建它：

```
$ mv cpu.pprof default.pgo$ go build -o markdown.withpgo
```
我们可以通过 go version 检查 PGO 是否在构建中被启用:

```
$ go version -m markdown.withpgo./markdown.withpgo: go1.21.0...        build   -pgo=/tmp/pgo121/default.pgo
```
## 评估

我们将使用负载生成器的 Go 基准版本来评估 PGO 对性能的影响。

首先，我们将对没有 PGO 的服务器进行基准测试。启动该服务器：

```
$ ./markdown.nopgo
```
当它运行时，运行几个基准迭代：

```
$ go get github.com/prattmic/markdown-pgo@latest$ go test github.com/prattmic/markdown-pgo/load -bench=. -count=40 -source $(pwd)/README.md > nopgo.txt
```
完成后，终止原始服务器并使用 PGO 启动该版本：

```
$ ./markdown.withpgo
```
当它运行时，运行几个基准迭代：

```
$ go test github.com/prattmic/markdown-pgo/load -bench=. -count=40 -source $(pwd)/README.md > withpgo.txt
```
完成后，让我们比较一下结果：

```
$ go install golang.org/x/perf/cmd/benchstat@latest$ benchstat nopgo.txt withpgo.txtgoos: linuxgoarch: amd64pkg: github.com/prattmic/markdown-pgo/loadcpu: Intel(R) Xeon(R) W-2135 CPU @ 3.70GHz        │  nopgo.txt  │            withpgo.txt             │        │   sec/op    │   sec/op     vs base               │Load-12   374.5µ ± 1%   360.2µ ± 0%  -3.83% (p=0.000 n=40)
```
新版本速度提高了约 3.8%！在 Go 1.21 中，启用 PGO 后工作负载的 CPU 使用率通常会提高 2% 到 7%。配置文件包含大量有关应用程序行为的信息，Go 1.21 刚刚开始通过使用这些信息进行一组有限的优化来探索表面。随着编译器的更多部分利用 PGO，未来的版本将继续提高性能。

# 后续步骤

在此示例中，收集配置文件后，我们使用原始构建中使用的完全相同的源代码重建了我们的服务器。在现实世界中，总是有持续的发展。因此，我们可以从生产中收集运行上周代码的配置文件，并使用它来构建今天的源代码。那完全没问题！Go 中的 PGO 可以毫无问题地处理源代码的微小更改。当然，随着时间的推移，源代码会越来越漂移，因此偶尔更新配置文件仍然很重要。

有关使用 PGO、最佳实践和注意事项的更多信息，请参阅 配置文件引导优化用户指南[5]。如果您对幕后发生的事情感到好奇，请继续阅读！

# 原理剖析

为了更好地理解是什么让这个应用程序变得更快，让我们深入了解一下性能有何变化。我们将看一下两种不同的 PGO 驱动的优化。

## 内联

为了观察内联改进，让我们分析这个带有和不带有 PGO 的 Markdown 应用程序。

我将使用一种称为差异分析的技术来对此进行比较，其中我们收集两个配置文件（一个带有 PGO，一个没有）并进行比较。对于差异分析，重要的是两个配置文件代表相同的工作量，而不是相同的时间量，因此我调整了服务器以自动收集配置文件，并将负载生成器调整为发送固定数量的请求，然后退出服务器。

我对服务器所做的更改以及收集的配置文件可以在https://github.com/prattmic/markdown-pgo找到。负载生成器使用 运行-count=300000 -quit。

作为快速一致性检查，让我们看一下处理所有 300k 请求所需的总 CPU 时间：

```
$ go tool pprof -top cpu.nopgo.pprof | grep "Total samples"Duration: 116.92s， Total samples = 118.73s (101.55%)$ go tool pprof -top cpu.withpgo.pprof | grep "Total samples"Duration: 113.91s， Total samples = 115.03s (100.99%)
```
CPU 时间从约 118 秒下降到约 115 秒，即约 3%。这与我们的基准结果一致，这是一个好兆头，表明这些配置文件具有代表性。

现在我们可以打开差异配置文件来寻找节省的空间：

```
$ go tool pprof -diff_base cpu.nopgo.pprof cpu.withpgo.pprofFile: markdown.profile.withpgo.exeType: cpuTime: Aug 28， 2023 at 10:26pm (EDT)Duration: 230.82s， Total samples = 118.73s (51.44%)Entering interactive mode (type "help" for commands， "o" for options)(pprof) top -cumShowing nodes accounting for -0.10s， 0.084% of 118.73s totalDropped 268 nodes (cum <= 0.59s)Showing top 10 nodes out of 668      flat  flat%   sum%        cum   cum%    -0.03s 0.025% 0.025%     -2.56s  2.16%  gitlab.com/golang-commonmark/markdown.ruleLinkify     0.04s 0.034% 0.0084%     -2.19s  1.84%  net/http.(*conn).serve     0.02s 0.017% 0.025%     -1.82s  1.53%  gitlab.com/golang-commonmark/markdown.(*Markdown).Render     0.02s 0.017% 0.042%     -1.80s  1.52%  gitlab.com/golang-commonmark/markdown.(*Markdown).Parse    -0.03s 0.025% 0.017%     -1.71s  1.44%  runtime.mallocgc    -0.07s 0.059% 0.042%     -1.62s  1.36%  net/http.(*ServeMux).ServeHTTP     0.04s 0.034% 0.0084%     -1.58s  1.33%  net/http.serverHandler.ServeHTTP    -0.01s 0.0084% 0.017%     -1.57s  1.32%  main.render     0.01s 0.0084% 0.0084%     -1.56s  1.31%  net/http.HandlerFunc.ServeHTTP    -0.09s 0.076% 0.084%     -1.25s  1.05%  runtime.newobject(pprof) topShowing nodes accounting for -1.41s， 1.19% of 118.73s totalDropped 268 nodes (cum <= 0.59s)Showing top 10 nodes out of 668      flat  flat%   sum%        cum   cum%    -0.46s  0.39%  0.39%     -0.91s  0.77%  runtime.scanobject    -0.40s  0.34%  0.72%     -0.40s  0.34%  runtime.nextFreeFast (inline)     0.36s   0.3%  0.42%      0.36s   0.3%  gitlab.com/golang-commonmark/markdown.performReplacements    -0.35s  0.29%  0.72%     -0.37s  0.31%  runtime.writeHeapBits.flush     0.32s  0.27%  0.45%      0.67s  0.56%  gitlab.com/golang-commonmark/markdown.ruleReplacements    -0.31s  0.26%  0.71%     -0.29s  0.24%  runtime.writeHeapBits.write    -0.30s  0.25%  0.96%     -0.37s  0.31%  runtime.deductAssistCredit     0.29s  0.24%  0.72%      0.10s 0.084%  gitlab.com/golang-commonmark/markdown.ruleText    -0.29s  0.24%  0.96%     -0.29s  0.24%  runtime.(*mspan).base (inline)    -0.27s  0.23%  1.19%     -0.42s  0.35%  bytes.(*Buffer).WriteRune
```
指定pprof -diff_base 时，pprof 中显示的值是两个配置文件之间的差异。例如，runtime.scanobject使用 PGO 时比不使用 PGO 时使用的 CPU 时间少 0.46 秒。另一方面，gitlab.com/golang-commonmark/markdown.performReplacements多使用了 0.36 秒的 CPU 时间。在差异分析中，我们通常希望查看绝对值（flat和cum列），因为百分比没有意义。

top -cum显示累积变化的最大差异。也就是说，函数和该函数的所有传递被调用者的 CPU 差异。这通常会显示程序调用图中的最外层框架，例如main或另一个 goroutine 入口点。在这里我们可以看到大部分节省来自ruleLinkify处理 HTTP 请求的部分。

top显示最大的差异仅限于函数本身的变化。这通常会显示程序调用图中的内部框架，其中大部分实际工作都在其中发生。在这里我们可以看到，个人储蓄主要来自runtime职能。

那些是什么？让我们查看一下调用堆栈，看看它们来自哪里：

```
(pprof) peek scanobject$Showing nodes accounting for -3.72s， 3.13% of 118.73s total----------------------------------------------------------+-------------      flat  flat%   sum%        cum   cum%   calls calls% + context----------------------------------------------------------+-------------                                            -0.86s 94.51% |   runtime.gcDrain                                            -0.09s  9.89% |   runtime.gcDrainN                                             0.04s  4.40% |   runtime.markrootSpans    -0.46s  0.39%  0.39%     -0.91s  0.77%                | runtime.scanobject                                            -0.19s 20.88% |   runtime.greyobject                                            -0.13s 14.29% |   runtime.heapBits.nextFast (inline)                                            -0.08s  8.79% |   runtime.heapBits.next                                            -0.08s  8.79% |   runtime.spanOfUnchecked (inline)                                             0.04s  4.40% |   runtime.heapBitsForAddr                                            -0.01s  1.10% |   runtime.findObject----------------------------------------------------------+-------------(pprof) peek gcDrain$Showing nodes accounting for -3.72s， 3.13% of 118.73s total----------------------------------------------------------+-------------      flat  flat%   sum%        cum   cum%   calls calls% + context----------------------------------------------------------+-------------                                               -1s   100% |   runtime.gcBgMarkWorker.func2     0.15s  0.13%  0.13%        -1s  0.84%                | runtime.gcDrain                                            -0.86s 86.00% |   runtime.scanobject                                            -0.18s 18.00% |   runtime.(*gcWork).balance                                            -0.11s 11.00% |   runtime.(*gcWork).tryGet                                             0.09s  9.00% |   runtime.pollWork                                            -0.03s  3.00% |   runtime.(*gcWork).tryGetFast (inline)                                            -0.03s  3.00% |   runtime.markroot                                            -0.02s  2.00% |   runtime.wbBufFlush                                             0.01s  1.00% |   runtime/internal/atomic.(*Bool).Load (inline)                                            -0.01s  1.00% |   runtime.gcFlushBgCredit                                            -0.01s  1.00% |   runtime/internal/atomic.(*Int64).Add (inline)----------------------------------------------------------+-------------
```
所以runtime.scanobject最终是来自于runtime.gcBgMarkWorker。Go GC 指南[6]告诉我们，这runtime.gcBgMarkWorker是垃圾收集器的一部分，所以runtime.scanobject节省的一定是 GC 节省的。还有nextFreeFast其他runtime功能呢？

```
(pprof) peek nextFreeFast$Showing nodes accounting for -3.72s， 3.13% of 118.73s total----------------------------------------------------------+-------------      flat  flat%   sum%        cum   cum%   calls calls% + context----------------------------------------------------------+-------------                                            -0.40s   100% |   runtime.mallocgc (inline)    -0.40s  0.34%  0.34%     -0.40s  0.34%                | runtime.nextFreeFast----------------------------------------------------------+-------------(pprof) peek writeHeapBitsShowing nodes accounting for -3.72s， 3.13% of 118.73s total----------------------------------------------------------+-------------      flat  flat%   sum%        cum   cum%   calls calls% + context----------------------------------------------------------+-------------                                            -0.37s   100% |   runtime.heapBitsSetType                                                 0     0% |   runtime.(*mspan).initHeapBits    -0.35s  0.29%  0.29%     -0.37s  0.31%                | runtime.writeHeapBits.flush                                            -0.02s  5.41% |   runtime.arenaIndex (inline)----------------------------------------------------------+-------------                                            -0.29s   100% |   runtime.heapBitsSetType    -0.31s  0.26%  0.56%     -0.29s  0.24%                | runtime.writeHeapBits.write                                             0.02s  6.90% |   runtime.arenaIndex (inline)----------------------------------------------------------+-------------(pprof) peek heapBitsSetType$Showing nodes accounting for -3.72s， 3.13% of 118.73s total----------------------------------------------------------+-------------      flat  flat%   sum%        cum   cum%   calls calls% + context----------------------------------------------------------+-------------                                            -0.82s   100% |   runtime.mallocgc    -0.12s   0.1%   0.1%     -0.82s  0.69%                | runtime.heapBitsSetType                                            -0.37s 45.12% |   runtime.writeHeapBits.flush                                            -0.29s 35.37% |   runtime.writeHeapBits.write                                            -0.03s  3.66% |   runtime.readUintptr (inline)                                            -0.01s  1.22% |   runtime.writeHeapBitsForAddr (inline)----------------------------------------------------------+-------------(pprof) peek deductAssistCredit$Showing nodes accounting for -3.72s， 3.13% of 118.73s total----------------------------------------------------------+-------------      flat  flat%   sum%        cum   cum%   calls calls% + context----------------------------------------------------------+-------------                                            -0.37s   100% |   runtime.mallocgc    -0.30s  0.25%  0.25%     -0.37s  0.31%                | runtime.deductAssistCredit                                            -0.07s 18.92% |   runtime.gcAssistAlloc----------------------------------------------------------+-------------
```
看起来nextFreeFast前 10 名中的其他一些最终来自runtime.mallocgc，GC 指南告诉我们的是内存分配器。

GC 和分配器成本的降低意味着我们总体分配的数量更少。让我们看一下堆配置文件以了解更多信息：

```
$ go tool pprof -sample_index=alloc_objects -diff_base heap.nopgo.pprof heap.withpgo.pprofFile: markdown.profile.withpgo.exeType: alloc_objectsTime: Aug 28， 2023 at 10:28pm (EDT)Entering interactive mode (type "help" for commands， "o" for options)(pprof) topShowing nodes accounting for -12044903， 8.29% of 145309950 totalDropped 60 nodes (cum <= 726549)Showing top 10 nodes out of 58      flat  flat%   sum%        cum   cum%  -4974135  3.42%  3.42%   -4974135  3.42%  gitlab.com/golang-commonmark/mdurl.Parse  -4249044  2.92%  6.35%   -4249044  2.92%  gitlab.com/golang-commonmark/mdurl.(*URL).String   -901135  0.62%  6.97%    -977596  0.67%  gitlab.com/golang-commonmark/puny.mapLabels   -653998  0.45%  7.42%    -482491  0.33%  gitlab.com/golang-commonmark/markdown.(*StateInline).PushPending   -557073  0.38%  7.80%    -557073  0.38%  gitlab.com/golang-commonmark/linkify.Links   -557073  0.38%  8.18%    -557073  0.38%  strings.genSplit   -436919   0.3%  8.48%    -232152  0.16%  gitlab.com/golang-commonmark/markdown.(*StateBlock).Lines   -408617  0.28%  8.77%    -408617  0.28%  net/textproto.readMIMEHeader    401432  0.28%  8.49%     499610  0.34%  bytes.(*Buffer).grow    291659   0.2%  8.29%     291659   0.2%  bytes.(*Buffer).String (inline)
```
-sample_index=alloc_objects选项向我们显示分配的数量，无论大小如何。这很有用，因为我们正在调查 CPU 使用率的下降，而这往往与分配计数而不是大小相关。这里有相当多的减少，但让我们关注最大的减少，mdurl.Parse。

作为参考，让我们看看没有 PGO 的情况下该函数的总分配计数：

```
$ go tool pprof -sample_index=alloc_objects -top heap.nopgo.pprof | grep mdurl.Parse   4974135  3.42% 68.60%    4974135  3.42%  gitlab.com/golang-commonmark/mdurl.Parse
```
之前的总计数是 4974135，这意味着mdurl.Parse已经消除了 100% 的分配！

让我们收集更多背景信息：

```
(pprof) peek mdurl.ParseShowing nodes accounting for -12257184， 8.44% of 145309950 total----------------------------------------------------------+-------------      flat  flat%   sum%        cum   cum%   calls calls% + context----------------------------------------------------------+-------------                                          -2956806 59.44% |   gitlab.com/golang-commonmark/markdown.normalizeLink                                          -2017329 40.56% |   gitlab.com/golang-commonmark/markdown.normalizeLinkText  -4974135  3.42%  3.42%   -4974135  3.42%                | gitlab.com/golang-commonmark/mdurl.Parse----------------------------------------------------------+-------------
```
调用 mdurl.Parse 的是来自 markdown.normalizeLink 和 markdown.normalizeLinkText。

```
(pprof) list mdurl.ParseTotal: 145309950ROUTINE ======================== gitlab.com/golang-commonmark/mdurl.Parse in /usr/local/google/home/mpratt/go/pkg/mod/gitlab.com/golang-commonmark/mdurl@v0.0.0-20191124015652-932350d1cb84/parse.go  -4974135   -4974135 (flat， cum)  3.42% of Total         .          .     60:func Parse(rawurl string) (*URL， error)          .          .     65:  -4974135   -4974135     66:   var url URL         .          .     67:   rest := rawurl         .          .     68:   hostless := false         .          .     69:   if n > 0  else 
```
也就是说，我们为最有可能出现的具体类型添加运行时检查，如果是这样，则使用具体调用，否则回退到标准间接调用。这里的优点是公共路径（使用*os.File）可以内联并应用额外的优化，但我们仍然保留后备路径，因为配置文件并不能保证始终如此。

在我们对 Markdown 服务器的分析中，我们没有看到 PGO 驱动的去虚拟化，但我们也只关注了受影响最大的区域。PGO（以及大多数编译器优化）通常会在许多不同地方进行非常小的改进的总和中产生好处，因此可能发生的事情不仅仅是我们所看到的。

内联和去虚拟化是 Go 1.21 中提供的两种 PGO 驱动的优化，但正如我们所见，这些通常会解锁额外的优化。此外，Go 的未来版本将继续通过额外的优化来改进 PGO。

### 参考资料

[1]https://go.dev/blog/pgo: https://go.dev/blog/pgo

[2]gitlab.com/golang-commonmark/markdown: https://pkg.go.dev/gitlab.com/golang-commonmark/markdown

[3]net/http/pprof: https://pkg.go.dev/net/http/pprof

[4]简单的程序: https://github.com/prattmic/markdown-pgo/blob/main/load/main.go

[5]配置文件引导优化用户指南: https://go.dev/doc/pgo

[6]GC指南: https://go.dev/doc/gc-guide#Identiying_costs

[7]mdurl.Parse: https://gitlab.com/golang-commonmark/mdurl/-/blob/bd573caec3d827ead19e40b1f141a3802d956710/parse.go#L60

[8]markdown.normalizeLink: https://gitlab.com/golang-commonmark/markdown/-/blob/fd7971701a0cab12e9347109a4c889f5c0a1a479/util.go#L53

[9]markdown.normalizeLinkText: https://gitlab.com/golang-commonmark/markdown/-/blob/fd7971701a0cab12e9347109a4c889f5c0a1a479/util.go#L68

