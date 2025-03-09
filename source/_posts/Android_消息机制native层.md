---
title: Android 消息机制（native层)
cover_image: /img/cover.png
abbrlink: eedcf486
date: 2023-04-19 08:10:39
---

书接上文：[Android 消息机制（Java层)](http://mp.weixin.qq.com/s?__biz=Mzg3MTYwNzEwMA==&mid=2247495045&idx=1&sn=2574144f8d14b3b4d0ab2a2d80556e24&chksm=cef953d6f98edac0f35a7188343c809c191162c09f6cdff2642cbe94de224939bb846faede8a&scene=21#wechat_redirect)native层中一些重要的类：

* MessageQueue：和java层中的MessageQueue同名，里面包含一个Looper
* NativeMessageQueue:是MessageQueue的继承类，是native层的消息队列，只是一个代理类，它的大部分逻辑在Looper中实现
* Looper：相当于java层的Handler，他可以发送消息，取出消息，处理消息
* MessageHandler：native层的消息处理类，Looper将消息处理的逻辑交给这个类
* WeakMessageHandler：是MessageHandler的继承类，也是native层的消息处理类，最终还是会把消息处理的逻辑交给MessageHandler
```
public final class MessageQueue 
```
这是java层中声明的一些native方法

## nativeInit()

MessageQueue在Looper的构造器中创建，在MessageQueue的构造器中会执行下面代码：

```
MessageQueue(boolean quitAllowed) 
```
在这里会执行native层的nativeInit方法，并将其返回值保存到mPtr中，下面再看一下native层中这个方法的实现：

```
static jlong android_os_MessageQueue_nativeInit(JNIEnv* env, jclass clazz) 
```
可以发现在这里创建了一个NativeMessageQueue对象，增加了其引用计数，并将这个对象的指针强制转换为Long类型返回，也就是说java层保存了native层nativeMessageQueue的指针，下面再看一下NativeMessageQueue的构造函数：

```
NativeMessageQueue::NativeMessageQueue()     : mPollEnv(NULL), mPollObj(NULL), mExceptionObj(NULL) }
```
在这个构造器中实现了线程中Looper的初始化

下面看一下native层中Looper的构造函数：

```
Looper::Looper(bool allowNonCallbacks) :        mAllowNonCallbacks(allowNonCallbacks), mSendingMessage(false),        mPolling(false), mEpollFd(-1), mEpollRebuildRequired(false),        mNextRequestSeq(0), mResponseIndex(0), mNextMessageUptime(LLONG_MAX) 
```
其中mWakeEventFd是文件描述符，在linux中打开或创建一个文件都会返回一个文件描述符，读写文件需要使用这个文件描述符来指定待读写的文件，所以文件描述符就是指代被打开的文件，所有文件的IO操作都需要这个文件描述符

```
void Looper::rebuildEpollLocked()     //2、创建一个新的epoll文件描述符，并注册wake管道    mEpollFd = epoll_create(EPOLL_SIZE_HINT);//EPOLL_SIZE_HINT为8    struct epoll_event eventItem;    memset(& eventItem, 0, sizeof(epoll_event)); //置空eventItem    //3、设置监听事件类型和需要监听的文件描述符    eventItem.events = EPOLLIN;//监听可读事件（EPOLLIN）    eventItem.data.fd = mWakeEventFd;//设置唤醒事件的fd（mWakeEventFd）    //4、将唤醒事件fd(mWakeEventFd)添加到epoll文件描述符(mEpollFd)，并监听唤醒事件fd(mWakeEventFd)    int result = epoll_ctl(mEpollFd, EPOLL_CTL_ADD, mWakeEventFd, & eventItem);           //5、将各种事件，如键盘、鼠标等事件的fd添加到epoll文件描述符(mEpollFd)，进行监听    for (size_t i = 0; i < mRequests.size(); i++)     }}}
```
在这个方法中使用了epoll机制，下面是对epoll机制的一些简单了解：

* 首先调用epoll_create方法来建立一个epoll对象(在epoll系统中为这个句柄分配空间)，
* 之后调用epoll_ctl向epoll对象中进行添加，删除，修改操作
* 最后调用epoll_wait收集所有事件的连接
epoll的优点简单来说就是可以显著减少大量并发连接中只有少部分活跃的CPU资源耗费，

那么这一部分总的来说就是以下几个步骤：

* java层中在Looper的构造函数中创建MessageQueue对象
* MessageQueue的构造器中又会调用native层的nativeInit方法初始化native层的NativeMessageQueue，在这个类的构造函数中又会创建Looper对象，并通过管道和epoll机制建立一套消息机制
* native层构建完毕后，会将native层的nativeMessageQueue对象转换为Long类型的数据储存到java层的MessageQueue的mPtr中
## nativePollOnce

这个方法的调用在messagequeue.next中

```
Message next()     //... }
```
next方法返回一个Message对象，在没有消息时nativePollOnce会进入阻塞

下面看一下这个函数的源码：

```
static void android_os_MessageQueue_nativePollOnce(JNIEnv* env, jobject obj,        jlong ptr, jint timeoutMillis) 
```
这个方法首先从java层中获取到保存的nativeMessageQueue对象，然后再调用pollOnce方法，在这个方法内部又调用了Looper.pollOnce方法，下面看一下这个方法的实现

```
int Looper::pollOnce(int timeoutMillis, int* outFd, int* outEvents, void** outData)         //处理内部轮询        result = pollInner(timeoutMillis);    }}
```
该方法也是一个死循环，如果result不等于0，那么就会返回到java层，那么看一下这个result的赋值操作，

```
int Looper::pollInner(int timeoutMillis)  else         } else     }        //2、下面是处理Native的Message    Done:;    mNextMessageUptime = LLONG_MAX;    //mMessageEnvelopes是一个Vector集合，它代表着native中的消息队列    while (mMessageEnvelopes.size() != 0)             mLock.lock();            mSendingMessage = false;            //result等于POLL_CALLBACK，表示某个监听事件被触发            result = POLL_CALLBACK;        } else     }    //释放锁    mLock.unlock();    //...}
```
这个方法内执行的逻辑大致可分为三步：

1. 执行epoll_wait方法，等待事件发生或超时
如果文件描述符监听的任何时间发生，那么epoll_wait就会监听到，并将事件放入到事件集合中，并返回发生的时间数目，timeOut就是从java层中传过来的nextPollTimeOutMilis,当值为-1时表示java层的消息队列中没有消息，会一直等待下去，当值为0时就会立即返回,另外epoll机制只会把发生了的事件放入到事件集合中。

1. 遍历事件集合，检测是哪个文件描述符发生了IO事件
2. 处理native层的message
这里面就有一个结构体：

```
class Looper : public RefBase         MessageEnvelope(nsecs_t u, const sp h, const Message& m) : uptime(u), handler(h), message(m) {}        nsecs_t uptime;        //收信人handler        sp handler;        //信息内容message        Message message;    };       //...}
```
native层中的message存放在messageEnvelop中，然后进入循环中，如果到达执行时间，就会调用MessageHandler的handleMessage方法处理消息

## nativeWake

这个方法执行在消息入队时

```
boolean enqueueMessage(Message msg, long when)     }}
```
这个方法最终会调用到native层的Looper.wake方法

```
void Looper::wake() 
```
这个方法其实是使用write方法向管道中写入字符inc,如果写入失败就重复写入，直到写入成功为止，这个字符起到一个通知作用，用来通知线程处理消息

java层和native层通过MessageQueue的JNI进行连接，从而使得MessageQueue既能处理java层又能处理native层

