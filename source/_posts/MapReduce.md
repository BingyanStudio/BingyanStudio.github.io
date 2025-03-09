---
title: MapReduce
cover_image: /img/cover.png
abbrlink: bc1fb66d
date: 2022-03-13 11:11:55
---

# MapReduce

## MapReduce是什么

### Google三驾马车

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0RrIaF1yukjahMhVltxKclI9yqt654th2F0x5IoniaM9tcbXP5TQwYqAfQ/640?wx_fmt=png)Google的基础设施有三驾马车，分别是《Google File System》、《Google MapReduce》以及《Google BigTable》

现代各种优秀的分布式系统实现的本质，都源于那几篇经典的研究和论文。比如非常流行的 Hadoop(HDFS、MapReduce、Hbase)、Spark、Hive，以及国产数据库软件 TiDB、OceanBase 都是参考这几篇经典论文设计的

三驾马车之间的关系：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0RrBp8u7RLR8LwVlDV3VnvuRlnN5pJQLGYMhaDpickeUFvpAyUvpndNvSQ/640?wx_fmt=png)### MapReduce解决的问题

谷歌在这为了处理海量的原始数据，已经实现了数以百计的、专用的计算方法。这些计算方法用来处理大量的原始数据。

2004年谷歌提出了MapReduce, 在此之前谷歌程序员面对的大规模数据集,常常需要编程实现:

1. 统计某个关键词出现的频率,计算PageRank
2. 对大规模数据按词频排序
3. 对多台机器上的文件进行Grep等
这些工作不可能在一台机器上完成(否则也不能称之为大规模)，要想在可接受的时间内完成运算，只有将这些计算分布在成百上千的主机上,因此谷歌的程序员每次编写代码都需要处理并行计算,分发数据,处理错误等问题。

为了解决上述复杂的问题，谷歌设计了一个新的抽象模型，使用这个抽象模型，我们只要表述我们想要执行的简单运算即可，而不必关心并行计算、容错、数据分布、负载均衡等复杂的细节，这些问题都被封装在了一个库里面。

设计这个抽象模型的灵感来自Lisp和许多其他函数式语言的Map和Reduce的原语，在大多数运算都包含这样的操作：

> 在输入数据的“逻辑”记录上应用Map操作得出一个中间key/value pair集合，然后在所有具有相同key值的value值上应用Reduce操作，从而达到合并中间的数据，得到一个想要的结果的目的。
> 
> 使用MapReduce模型，再结合用户实现的Map和Reduce函数，我们就可以非常容易地实现大规模并行化计算；通过MapReduce模型自带的“再次执行”（re-execution）功能，也提供了初级的容错实现方案

## MapReduce的架构/实现

### MapReduce编程模型

例子：WordCount

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0Rr6iblhvxHesBHL4C2vToQtiaRGkDOX1XvBkWRibgXmkbBZA3FY3tLx888g/640?wx_fmt=png)利用一个输入的key/value pair集合来产生一个输出的key/value pair集合。

MapReduce库的用户用两个函数表达这个计算：Map和Reduce。

用户自定义的Map函数接受一个输入的key/value pair集合，然后产生一个中间key/value pair值的集合。MapReduce库把所有具有相同中间key值I的中间value值集合在一起后传递给Reduce函数。

用户自定义的Reduce函数接受一个中间key的值I和相关的一个value值的集合。Reduce函数合并这些value值，形成一个较小的value值的集合。通常我们通过一个迭代器把中间value值提供给Reduce函数，这样我们就可以处理无法全部放入内存中的大量的value值的集合。

需要一提的是：每个Map在结束后，会将输出结果产生为一个中间文件

```
map(String key, String value):    // key: document name    // value: document contents    for each word w in value:      EmitIntermediate(w, "I");reduce(String key, Iterator values):    // key: a word    // values: a list of counts    int result = 0;    for each v in values:        result += ParseInt(v);    Emit(AsString(result));      
```
用户提供的 Map 函数和 Reduce 函数应有如下类型：

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0RrznkURooWGUWOUl1om7K9UsvAmU4PlQgwcUt5vv8GPWAozMTpeo4iaFQ/640?wx_fmt=png)值得注意的是，在实际的实现中 MapReduce 框架使用 Iterator 来代表作为输入的集合，主要是为了避免集合过大，无法被完整地放入到内存中

### MapReduce的应用

分布式的Grep：Map函数输出匹配某个pattern的一行，Reduce函数是一个恒等函数，即把中间数据复制到输出。

计算URL访问频率：Map函数处理日志中web页面请求的记录，然后输出(URL,1)。Reduce函数把相同URL的value值都累加起来，产生(URL,记录总数)结果。

倒排索引：Map函数分析每个文档输出一个(word,document_id)的列表，Reduce函数的Input是一个word的所有(word，document_id)，排序所有的document_id，输出(word,list(document_id))。所有的输出集合形成一个简单的倒排索引，它以一种简单的算法跟踪词在文档中的位置。

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0Rrib9qicZrjSQSdGOnRDv9pUnvT28sPJBT9BEHuQevfrS90fEHw7u1cib4A/640?wx_fmt=png)### MapReduce架构

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0RrdRiaRCzgKK91icx5xYGM68lciaJmMSFGf38Wt8rYA9CGkU28yowzNbkhw/640?wx_fmt=png)1. 用户程序首先调用 MapReduce 库将输入文件分成个数据片段，每个数据片段的大小一般从16MB到64MB(可以通过可选的参数来控制每个数据片段的大小)。然后用户程序在机群中创建大量的程序副本。

2. 这些程序副本中有一个特殊的程序：master。副本中其它的程序都是worker程序，由master分配任务。有个Map任务和个Reduce任务将被分配，master将一个Map任务或Reduce任务分配给一个空闲的worker。每个worker一项任务

3. 被分配了Map任务的worker程序读取相关的输入数据片段，从输入的数据片段中解析出key/value pair，然后把key/value pair传递给用户自定义的Map函数，由Map函数生成并输出的中间key/value pair，并缓存在内存中。

4. 缓存中的key/value pair通过分区函数分成R个区域，之后周期性的写入到本地磁盘上。缓存的key/value pair在本地磁盘上的存储位置将被回传给master，由master负责把这些存储位置再传送给Reduce worker。

5. 当Reduce worker程序接收到master程序发来的数据存储位置信息后，使用RPC(Remote Procedure Call：该协议允许运行于一台计算机的程序调用另一个地址空间(通常为一个开放网络的一台计算机)的子程序)从Map worker所在主机的磁盘上读取这些缓存数据。

当Reduce worker读取了所有的中间数据后，通过对key进行排序后使得具有相同key值的数据聚合在一起。由于许多不同的key值会映射到相同的Reduce任务上，因此必须进行排序。如果中间数据太大无法在内存中完成排序，那么就要在外部进行排序。

6. Reduce worker程序遍历排序后的数据，对于每一个唯一的key值，Reduce worker程序将这个key值和它相关的value值的集合传递给用户自定义的Reduce函数。Reduce函数的输出被追加到所属分区的输出文件。

7. 当所有的Map和Reduce任务都完成之后，master唤醒用户程序。此时，用户程序中的MapReduce调用返回到用户代码中。

在这个过程中：

master应当维护多种数据结构。它会存储每个map和reduce任务的状态（空闲、处理中、完成），和每台工作机器的ID（对应非空闲的任务）。

master是将map任务产生的中间文件的位置传递给reduce任务的通道。因此，master要存储每个已完成的map任务产生的个中间文件的位置和大小。map任务完成时，master会接收到位置和大小的更新信息。这些信息会被逐步发送到正在处理中的reduce任务节点处。

## MapReduce的容错

因为MapReduce库的设计初衷是使用由成百上千的机器组成的集群来处理超大规模的数据，所以，这个库必须要能很好的处理机器故障。

### worker故障

如何检测worker的故障？发心跳包。

master周期性的ping每个worker。如果在一定时间范围内没有收到worker返回的信息，master将把这个worker标记为失效。

所有由这个失效的worker完成的Map任务被重设为初始的空闲状态，之后这些任务就可以被安排给其他的worker。同样的，worker失效时正在运行的Map或Reduce任务也将被重新置为空闲状态，等待重新调度。

值得一提的是：worker完成的Reduce任务的输出存储在全局文件系统上，因此不需要再次执行。而Map任务的输出存储在worker对应的这台机器上，不可访问。

当一个Map任务首先被worker A执行，之后由于worker A失效了又被调度到worker B执行，这个“重新执行”的动作会被通知给所有执行Reduce任务的worker。任何还没有从worker A读取数据的Reduce任务将从worker B读取数据

通过这样的方式，MapReduce可以处理大规模worker失效的情况。

### master故障

那就寄！

备份。一个解决办法是让master周期性将其维护的数据写入磁盘，作为checkpoint，当master挂了后可以通过checkpoint来重启master进程。

但需要考虑到master故障后再恢复比较麻烦，只有一个master进程，Google的实现是当master故障后，让客户检查到这个状态，然后中止运算并重启MapReduce操作。

## MapReduce在实践过程中的优化

### 存储位置

在分布式系统中，网络带宽通常是稀缺资源，很容易成为系统瓶颈。为了减少网络通信，Google通过将输入数据存储在本地磁盘上。

1. GFS把每个文件按64MB一个Block分隔，每个Block保存在多台机器上，环境中就存放了多份拷贝(一般是3个拷贝)。MapReduce的master在调度Map任务时会考虑输入文件的位置信息，尽量将一个Map任务调度在包含相关输入数据拷贝的机器上执行
2. 如果无法实现，就尝试在保存有输入数据拷贝的机器附近的机器上执行Map任务（比如在同一个交换机的机器上执行Map）
### 任务粒度

正常情况下、比集群中的worker数量大很多，这样一来在每台worker机器都执行大量的不同任务能够提高集群的负载均衡能力

但实际情况下，master会执行次调度，并且在内存中保存个状态

的值是由用户指定的，而的值也尽量满足每一个独立任务都是处理大约16M到64M的输入数据，使得上述存储位置的优化最有效

比如，使用2000个worker

### 备用任务

在运算过程中，如果有几台机器运行特别慢，会极大影响MapReduce操作的总时间。

这里有个通用的方法。MapReduce会启用处理中状态(in-progress)的任务的备用任务进程，它和原任务做相同的事情，无论哪个完成了任务，都视为该任务已经完成

### 分区函数

前面说到的值是由用户指定的，可以通过分区函数对数据分区，比如用hash(key) mod R进行分区，能够让key/value pair比较均衡的分给执行Reduce的worker。当输出的key值是URLs，我们希望每个主机的所有条目保持在同一个输出文件中，又可以通过hash(hostname(urlkey)) mod R将所有来自同一个主机的URLs分在一个分区中。

### 顺序保证

我们确保在给定的分区中，中间key/value pair数据的处理顺序是按照key值增量顺序处理的。这样的顺序保证对每个分区生成一个有序的输出文件，这对于需要对输出文件按key值随机存取的应用非常有意义。

### 合并函数

比如在进行词频统计时，一些词的出现频率非常的高，这会导致产生例如(I,1)这种key/value pair，导致中间文件到同一个Reduce的大小不均衡。

此时可以通过指定一个可选的combiner函数，combiner函数首先在本地将这些记录进行一次合并，然后将合并的结果再通过网络发送出去。

## Shuffle过程/细节

上面的优化大部分都是在优化Shuffle这一步骤

shuffle需要对Map的输出进行进一步整理并交给Reduce，这个过程会消耗大量的I/O资源，以及进行大量网络通信

Shuffle过程包含在Map和Reduce两端，即Map shuffle和Reduce shuffle

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0RrBUoaYMfRsR0nicIqIBG8HJJJibkE6UnKicqMOma6cicv93wGUSU3b2TA2w/640?wx_fmt=png)### Map Shuffle

在Map端的shuffle过程是对Map的结果进行分区、排序、分割，然后将属于同一划分（分区）的输出合并在一起并写在磁盘上，最终得到一个分区有序的文件，分区有序的含义是map输出的键值对按分区进行排列，具有相同partition值的键值对存储在一起，每个分区里面的键值对又按key值进行升序排列

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0RryfWST0cwcCkNW9MSQEFq7t7W4TDPoUXxI2jhc37WKwytI5hPpqI3dg/640?wx_fmt=png)Partition

即用分区函数处理

Collector

即用合并函数处理

值得一提的是，每个Map任务不断地将键值对输出到在内存中构造的一个环形数据结构中。使用环形数据结构是为了更有效地使用内存空间，在内存中放置尽可能多的数据。

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8SicNCjgr2zORMmP7VS1U0RribicNLhspUncsX8kt8jnbtotpb9HeCMKdBUibwhxJSS7EicblPEicmH0jMA/640?wx_fmt=png)### Reduce shuffle

Copy

Reduce任务通过HTTP向各个Map任务拖取它所需要的数据，这由master调度

Merge Sort

Map的输出数据已经是有序的，Merge进行一次合并排序，所谓Reduce端的sort过程就是这个合并的过程

## 参考

MapReduce: Simplified Data Processing on Large Clusters https://research.google/pubs/pub62/

MapReduce论文导读 https://hardcore.feishu.cn/docs/doccnxwr1i2y3Ak3WXmFlWLaCbh#

MapReduce Shuffle深入理解 https://zhuanlan.zhihu.com/p/36253130

MapReduce shuffle过程详解 https://blog.csdn.net/u014374284/article/details/49205885

