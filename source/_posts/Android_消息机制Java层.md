---
title: Android 消息机制（Java层)
cover_image: /img/cover.png
abbrlink: c55071d
date: 2022-10-16 15:16:52
---

![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8RImZC9gWQicbNE9dG9mOfpqNRa1JkXvXgnv0ZgO759Hr6Zkicp9BYA4XHRgxna37XibQjt6tTRmDFlA/640?wx_fmt=png)## ThreadLocal

`ThreadLocal`与同步机制相反，用于保证多线程情况下各个线程数据的独立性，那么这是如何实现的呢？观察`ThreadLocal`的`set`方法：```
public void set(T value) 
```
会发现在这个方法中首先会得到当前的线程，然后获取到`ThreadLocalMap`对象，如果这个对象不为空，就将数据保存到这个`map`中，否则就重新创建一个`map`对象，并将数据保存到这个`map`中，这个`ThreadLocalMap`相当于一个`HashMap`，用于保存这个线程中的数据。然后再看它的`get`方法：```
public T get()     }    return setInitialValue();}
```
这个方法跟`set`方法相同，也是先获取到这个线程的`ThreadLocalMap`对象，并判断是否为空，如果为空就返回初始值并保存初始值，否则就返回之前存放的数据。其中`setInitialValue`就是获取初始值并保存的方法可以通过`remove`方法来移除`ThreadLocal`对应的值查看`Thread`的源码发现每一个`Thread`对象都有一个`ThreadLocalMap`对象用于保存当前线程的数据，这就保证了多线程之间不会相互影响。下面再来看一下`ThreadLocalMap`,先来看一下构造方法：```
ThreadLocalMap(ThreadLocal firstKey, Object firstValue) 
```
在构造方法中先创建了一个数组，并将第一次的数据存储到这个数组中。```
// 初始容量，必须是 2 的幂private static final int INITIAL_CAPACITY = 16;// 存储数据的哈希表private Entry[] table;// table 中已存储的条目数private int size = 0;// 表示一个阈值，当 table 中存储的对象达到该值时就会扩容private int threshold;// 设置 threshold 的值private void setThreshold(int len) 
```
其中`Entry`是`ThreadLocalMap`中的一个内部类```
static class Entry extends WeakReference<ThreadLocal> }
```
下面看一下`ThreadLocalMap`的`set`方法：```
private void set(ThreadLocal key, Object value)         // 若索引位置的 Entry 的 key 为 null（key 已经被回收了），表示该位置的 Entry 已经无效，用要保存的键值替换该位置上的 Entry        if (k == null)     }    // 要存放的索引位置没有 Entry，将当前键值作为一个 Entry 保存在该位置    tab[i] = new Entry(key, value);    // 增加 table 存储的条目数    int sz = ++size;    // 清除一些无效的条目并判断 table 中的条目数是否已经超出阈值    if (!cleanSomeSlots(i, sz) && sz >= threshold)        rehash(); // 调整 table 的容量，并重新摆放 table 中的 Entry}
```
在这个方法中首先用当前`ThreadLocal`的`threadLocalHashCode`来计算索引值（每创建一个`ThreadLocal`都会生成一个相应的`threadLocalHashCode`值```
private final int threadLocalHashCode = nextHashCode();private static AtomicInteger nextHashCode =    new AtomicInteger();// 每次创建 ThreadLocal 对象是 HashCode 的增量private static final int HASH_INCREMENT = 0x61c88647;// 计算 ThreadLocal 对象的 HashCodeprivate static int nextHashCode() 
```
注意使用了`AtomicInteger`来保证不同线程状态下的同步在保存数据时，如果索引的位置有`Entry`且`Entry`的`key`为`null`由于`Entry`保存数据的方式是用的弱引用，就会将`Entry`进行回收，下面是获取`Entry`：```
private Entry getEntry(ThreadLocal key) 
```
因为可能存在哈希冲突，所以可能需要调用`getEntryAfterMiss`来获取```
private Entry getEntryAfterMiss(ThreadLocal key, int i, Entry e)     return null;}
```
## 一些准备工作

首先看一下`Handler`的构造器，`Handler`的构造器有很多，下面看最关键的一个```
public Handler(Callback callback, boolean async)     mQueue = mLooper.mQueue;    mCallback = callback;    mAsynchronous = async;}
```
会发现先获取`looper`对象，然后判断是否为空，也就是说在使用这个构造器的时候必须先调用`Looper.prepare`这个方法，否则就会抛出异常接着再看一下`Looper`的构造方法```
private Looper(boolean quitAllowed) 
```
发现这是一个私有方法，接着再来看一下他是在哪调用的呢？```
public static void prepare() private static void prepare(boolean quitAllowed)     sThreadLocal.set(new Looper(quitAllowed));}
```
从这里就可以看出来为什么要在一个线程中构造`Handler`之前必须调用`prepare`方法了，并且这个`Looper`对象放在了当前线程的私有内存中，主线程中初始化`Looper`对象的逻辑基本相同```
public static void prepareMainLooper()         sMainLooper = myLooper();    }}
```
并且系统已经帮我们在`ActivityThread`类的`main`方法中调用了这个方法```
public static void main(String[] args)     ...    Looper.loop();    throw new RuntimeException("Main thread loop unexpectedly exited");}
```
这里面的`loop`方法是一个死循环，会一直调用`MessageQueue`的`next`方法获取`Message`，然后交由`Hander`处理初始化工作就这些，下面就是发送消息的环节## 发送消息

下面先看一下`send`方法```
//最终会调用sendMessageDelayedpublic final boolean sendMessage(Message msg)public final boolean sendMessageDelayed(Message msg, long delayMillis)    //注意对应的执行时间    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);}//最终会调用sendEmptyMessageDelayedpublic final boolean sendEmptyMessage(int what)public final boolean sendEmptyMessageDelayed(int what, long delayMillis) 
```
会发现`sendEmptyMessage`其实仍然是包装成了一个`Message`对象,下面再看一下`post`的一系列方法```
public final boolean post(Runnable r)public final boolean postDelayed(Runnable r, long delayMillis)// 将Runnable转为Messageprivate static Message getPostMessage(Runnable r) 
```
会发现最终这些`send`和`post`方法都会指向`sendMessageAtTime`这个方法，下面来看一下这个方法```
public boolean sendMessageAtTime(Message msg, long uptimeMillis)     return enqueueMessage(queue, msg, uptimeMillis);}
```
在这个方法中首先会判断`MessageQueue`对象是否为空，不为空的话就会调用`enqueueMessage`方法```
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis)     // MessageQueue的消息入队    return queue.enqueueMessage(msg, uptimeMillis);}
```
这里会将`message`对象的`target`设置为当前的`Handler`对象，并且执行消息入队操作## 消息入队

```
boolean enqueueMessage(Message msg, long when)  else                 if (needWake && p.isAsynchronous())             }            msg.next = p; // invariant: p == prev.next            prev.next = msg;        }        // We can assume mPtr != 0 because mQuitting is false.        // mPtr是native层的MessageQueue的引用地址，是在MessageQueue的构造方法里初始化的        // 这样便可以将native层和java层的对象关联起来        if (needWake)     }    return true;}
```
这个方法会根据`Message`对象的`when`来找到`message`的位置并插入到队列中，这个队列其实是一个单向链表### 同步屏障

它的主要作用就是及时的更新UI，如果主线程中消息队列中除更新UI外还有其他的消息，那么就会先执行更新UI的操作```
public int postSyncBarrier() private int postSyncBarrier(long when)         }        if (prev != null)  else         return token;    }}
```
发现也是根据`message`的`when`来进行消息的入队，但是并没有给这个`message`对象设置`target`同步屏障的作用就是屏蔽消息队列中该同步屏障之后的所有同步消息，只处理异步消息，保证异步消息优先执行,可以调用`message`的`setAsynchronous`来设置同步或异步消息，也可以通过`handler`构造函数中的参数`async`来设置，主要应用于UI的绘制## Looper循环和消息出队

```
public static void loop()     final MessageQueue queue = me.mQueue;    ...    for (;;)         ...        try  finally         }        ...                // Message回收        msg.recycleUnchecked();    }}
```
开启一个死循环，在这个循环中获取消息队列中的消息，并进行消息的处理，下面看一下`next`方法```
Message next()     // 用于统计当前闲置Handler数量    int pendingIdleHandlerCount = -1; // -1 only during first iteration    // 阻塞的时长    int nextPollTimeoutMillis = 0;    for (;;)         // 实现阻塞，阻塞时长为nextPollTimeoutMillis，Looper.loop()方法中的might block就是来自这里        nativePollOnce(ptr, nextPollTimeoutMillis);        synchronized (this)  while (msg != null && !msg.isAsynchronous());            }            if (msg != null)  else  else                     msg.next = null;                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);                    msg.markInUse();                    return msg;                }            } else             // Process the quit message now that all pending messages have been handled.            if (mQuitting)             // 消息队列为空或Message未到执行时间时，则开始处理IdleHandler            // If first time idle, then get the number of idlers to run.            // Idle handles only run if the queue is empty or if the first message            // in the queue (possibly a barrier) is due to be handled in the future.            if (pendingIdleHandlerCount < 0                    && (mMessages == null || now < mMessages.when))             if (pendingIdleHandlerCount <= 0)             if (mPendingIdleHandlers == null)             mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);        }        ...        // Reset the idle handler count to 0 so we do not run them again.        pendingIdleHandlerCount = 0;        // While calling an idle handler, a new message could have been delivered        // so go back and look again for a pending message without waiting.        nextPollTimeoutMillis = 0;    }}
```
这个方法也是一个死循环，会进行各种判断，如果消息执行时间还没到，就会设置阻塞时长，另外会判断是否消息屏障，并执行相应的操作## 消息处理

观察`handler`的`dispatchMessage````
public void dispatchMessage(Message msg)  else         }        handleMessage(msg);    }}private static void handleCallback(Message message) 
```
其中`msg.callback`是在`post`系列方法中设置的，而`mCallBack`是在构造器中传入的，其中的`handleMessage`默认实现为空如果返回`true`的话表示消息处理结束，不会再继续分发消息，否则会分发给`handler`的`handleMessage`处理然后结束## 移除消息

```
public final void removeMessages(int what) // MessageQueue.javavoid removeMessages(Handler h, int what, Object object)     synchronized (this)         // Remove all messages after front.        while (p != null)             }            p = n;        }    }}
```
这利用了两个`while`循环，前面用来移除从队列头开始移除满足条件了`message`之后用来移除之后的`message`![](https://mmbiz.qpic.cn/mmbiz_png/rxLIic6e5g8TtibyNyK6KN8b7BSic0H9SajiaeNxUcawUSmAwSZ4CAI8jh7vnplqNYLHeBXBuV2jiblMQQySyz4iaN3w/640?wx_fmt=png)

