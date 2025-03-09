---
title: 浅析安卓IPC机制
cover_image: /img/cover.png
abbrlink: b8b59aae
date: 2024-05-10 15:26:32
---

IPC：Inter-Process-Communication，跨进程通信。在这里，需要区别开进程和线程。

一个应用可以有多个进程，一个进程可以有多个线程

## 安卓中的多进程模式

* 开启多进程

要开启多进程的模式只有一种，那就是使用android:process属性，属性内的值是进程名。以":romote"这种形式命名，是省略写法，完全进程名是<包名:remote>，并且这是一个私有线程

* 多进程会造成以下几个方面的问题

* 静态成员和单例模式完全失效
* 线程同步机制完全失效
* SharedPreferences的可靠性下降
* Application会多次创建
* 多进程开发的本质相当于是两个不同的应用采用了SharedUID模式

## IPC基础概念

#### Serializable接口

Serializable接口是一个空接口，为对象提供标准的序列化和反序列化操作。

其中比较需要注意的就是要手动指定serialVersionUID的值

#### Parcelable接口

基本使用如下：

```
class User(    val id: Int,    val name: String?) : Parcelable    override fun describeContents(): Int     override fun writeToParcel(dest: Parcel, flags: Int)     companion object CREATOR : Parcelable.Creator         override fun newArray(size: Int): Array     }}
```
Parcel内部包装了可序列化的数据，可以在Binder中自由传输

* 序列化功能由writeToParcel方法完成
* 反序列化功能由CREATOR来完成，其内部标明了如何创建序列化对象和数组，并通过Parcel一系列的read方法来完成反序列化过程
* 内容描述功能由describeContents来完成，大部分情况下返回0即可，只有当对象中存在文件描述符时，此方法返回1
Serializable使用场景更广泛，更便捷，但效率更低

Parcelable较为麻烦，但适应安卓场景，更为高效

#### Binder

* 从直观来说Binder是Android中的一个类，实现IBinder接口

* 从IPC角度来说，Binder是Android中跨进程通信的方式

* 从底层来说，Binder还可以是一种虚拟的物理设备，它的设备驱动是/dev/binder，这种通信方式是安卓独属，在Linux里没有

* 从Android Framwork角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager等等）和相应的ManagerService的桥梁

* 从Android应用层来说，BInder是客户端和服务端进行通信的媒介，当binderService时，客户端就可以获取服务端提供的服务或数据，这里的服务包括普通服务和基于AIDL的服务

可以看另一篇文章：Binder源码分析

其中有一个需要注意的，当客户端和服务端都位于一个进程时，是不会走跨进程的transact过程的，如果在不同进程则会需要走这样一个transact过程：

接下来是AIDL中Stub（服务端）和Proxy（客户端）的每个方法的含义

* DESCRIPTOR：Binder的唯一标识，一般用Binder的带包名类名表示
* asInterface(android.os.IBinder obj)：用于将服务端的Binder对象转换为客户端所需的AIDL接口类型对象。如果是同一进程，则返回Stub对象本身，否则返回系统封装后的Stub.proxy对象
* asBinder：此方法用于返回当前Binder对象
* onTransact：这个方法运行在服务端中的Binder线程池中，当客户端发起跨进程请求，会通过系统底层封装后交由此方法来处理。该方法原型为public Boolean onTransact(int code,android.os.Parcel data,android.os.Parcel reply,int flages)。服务端通过code可以确定客户端所请求的目标方法是什么，接着从data中取出目标方法的参数，然后执行目标方法，执行完毕后向reply写入返回值
* Proxy#getBookList：这个方法运行在客户端，当客户端调用此方法时：首先，创建该方法所需要的输入型Parcel对象_data，输出型对象_reply，和返回值对象List，接着调用transact发起RPC（远程过程调用）请求，同时将当前线程挂起，等待返回
* Proxy#addBook：同上
综上，我们需要额外注意两个点：

* 客户端发起请求时，当前线程会被挂起至服务端返回数据。所以如果一个远程方法是耗时的，那么不能在UI线程中发起。

* 由于服务端的Binder运行在Binder线程池中，所以不管是否耗时都应该采用同步的方式去实现

这里还有一个小技巧：

Binder有两个很重要的方法，linkToDeath和unlinkToDeath，用于服务端死亡后，Binder获取连接断裂信息

linkToDeath有两个参数，一个DeathRecipient对象，用于回调死亡后的操作，一个标记位，直接设置为0即可

使用方法：

```
//声明一个DeathRecipient对象val mDeathRecipient = IBinder.DeathRecipientmService = IMessageBoxManager.Stub.asInterface(binder)binder.linkToDeath(mDeathRecipient,0)
```
## Android中的IPC方式

具体方式有很多：

* 在Intent中附加extras
* 通过共享文件夹
* 采用Binder
* ContentProvider
* 网络通信Sockt
### 使用Bundle

四大组件哪个的三大组件（Activity，Service、Receive）都是支持在Intent中传递Bundle数据的

Bundle支持基本数据类型、实现了Parcelable接口、Serializable接口的对象以及一些Android支持的特殊对象

Bundle不支持的则无法传输

除了直接传递数据这种典型的使用场景，它还有一种特殊的使用场景，比如A进程在进行一个运算，完成后将结果传输给进程B，但Bundle不支持这种类型

可以考虑以下方法：通过Intent启动进程B的一个Service组件，让Service在后台进行计算，计算完毕后再启动B进程中真正要启动的目标组件。

这样的核心思想是，将A进程的计算任务转移到B进程的后台Service，可以避免进程通信问题

### 使用文件共享

两个进程共同读写一个文件，可以序列化一个对象到文件系统，再从另一个进程恢复这个对象。

要尽量避免并发读写的问题。

这种方式适合在对数据同步要求不高的进程之间通信。

SharePreference是类似的，不过系统对读写有一定的缓存测量

因而它不适合多进程模式

### 使用Messenger

Messenger是一种轻量级的IPC方案，底层实现是AIDL，看其构造方法的实现便可看出来

```
public Messenger(IBinder target) public Messenger(Handler target) 
```
Messenger的使用方法很简单，对AIDL做了封装，使得我们可以更简便地进行进程间的通信。同时，由于它一次处理一个请求，因此在服务端不用考虑线程同步的问题，这里是因为服务端中不存在并发执行的情形

实现一个Messenger有如下几个步骤，分为服务端和客户端

1. 服务端进程

首先，我们需要在服务端创建一个Service来处理客户端的连接请求，同时创建一个Handler并通过它来创建一个Messenger对象，然后在Service的onBind中返回这个Messenger对象底层的Binder

2. 客户端进程

客户端进程中，首先要绑定服务端Service，绑定成功后用服务端返回的IBinder对象创建一个Messenger对象。如果需要服务端能够回应客户端，就和服务端一样，我们还需要创建一个Handler并创建一个新的Messenger，并把这个Messenger对象通过Message的relpyTo参数传递给服务端，服务端通过这个replyTo参数可以回应客户端

使用方法如下：

```
const val MEG_FROM_CLIENT = 1const val MSG_FROM_SERVICE = 2class MessengerService : Service() ")                        val client = msg.replyTo                        val replyMessage = Message.obtain(null, MSG_FROM_SERVICE)                        val bundle = Bundle()                        bundle.putString("reply","嗯，你的消息我已经收到，稍后回复你")                        replyMessage.data = bundle                        client.send(replyMessage)                    }                    else ->                 }            }        }        val mMessenger = Messenger(MessengerHandler())    }    override fun onBind(intent: Intent?): IBinder? }class MessengerActivity : AppCompatActivity()                 else ->             }        }    }    private val mConnection = object :ServiceConnectioncatch (e:RemoteException)        }        override fun onServiceDisconnected(name: ComponentName?)     }    override fun onCreate(savedInstanceState: Bundle?)         val intent = Intent(this,MessengerService::class.java)        bindService(intent,mConnection,Context.BIND_AUTO_CREATE)    }    override fun onDestroy() }
```
总的来说，无论是客户端还是服务端，处理对方传来的消息都是用Handler。客户端远程绑定一个Messenger后，使用message的date向服务端发消息，若要接收回复，则用Handler创建一个Messenger，作为message的replyto参数传过去，服务端获取replyTo，并用这个Messagener向客户端发消息。

具体的流程图如下：

### 使用AIDL

AIDL的概念在上面已经有说明，使用AIDL大概分以下四步：

1. 服务端：

服务端首先要创建一个Service来监听客户端的连接请求，然后创建一个AIDL文件，将暴露给客户端的这个接口在这个AIDL文件中声明，最后在Service中实现这个AIDL接口

2. 客户端：

首先需要绑定服务端的Service，绑定成功后，将服务端返回的Binder对象转成AIDL接口所属的类型，接着就可以使用了

3. AIDL接口的创建

需要注意的是，不是所有数据类型在AIDL都是可以使用的，能用的有基本数据类型、String、List、Map、Parcelable、AIDL接口等，都可以作为数据传输

以下是一个创建的例子，具体的语法可以自行去查阅

// IMyAidlInterface.aidlpackage com.example.test;import com.example.test.User;// Declare any non-default types here with import statementsinterface IMyAidlInterface parcelable User;
4. 远程服务端Service的实现

定义AIDL接口后，接下来我们需要实现这个接口。以下是对上述接口的一个实现例子：

package com.example.test;import android.app.Service;import android.content.Intent;import android.os.IBinder;import android.os.RemoteException;import android.util.Log;import androidx.annotation.Nullable;public class RemoteService extends Service         @Override        public void addUser(User user) throws RemoteException     };    @Nullable    @Override    public IBinder onBind(Intent intent) }
5. 客户端的实现

客户端的实现比较简单，首先要绑定远程服务，绑定成功后将服务端返回的Binder对象转换成AIDL接口，然后就可以通过这个接口去调用服务端的远程方法了。下面是一个例子：

class MainActivity : AppCompatActivity() catch(e : RemoteException)        }        override fun onServiceDisconnected(name: ComponentName?)     }    override fun onCreate(savedInstanceState: Bundle?)         val intent = Intent().setComponent(ComponentName(            "com.example.test",            "com.example.test.RemoteService"        ))        bindService(intent,mConnection,Context.BIND_AUTO_CREATE)    }}
这是基本的使用流程，除此之外，还有一些使用技巧

* 如果服务端的Binder意外终止了怎么办？

要监听Binder的死亡，可以用IBinder.DeathRecipient类或者Connection的回调onServiceDisconnected，两者的区别是，前者的回调是运行在客户端的binder线程池，后者是直接在UI线程上

* 如何实现跨进程的Observer模式？

AIDL接口同样可以作为参数传递给服务端，只需要传递一个回调接口给服务端，供其注册，待需要更新时，遍历所有注册的AIDL接口，发送请求调用其方法。

这里要注意，由于跨进程时对象反序列化会创建一个新的对象，所以无法直接再次传递AIDL接口来取消注册，比较常用的做法是使用RemoteCallbackList，它能够通过AIDL底层的Binder去删除AIDL接口，而不仅仅是判断是否是新对象

示例代码如下，需要注意的是在mListenerList不是真正的List，遍历时必须要使用下面的固定格式，beginBroadcast和finishBroadcast一定要配对使用：

private val mListenerList = RemoteCallbackList()public fun register(action : OnEventListener)public fun unRegister(action : OnEventListener)public fun doAction()    }    mListenerList.finishBroadcast()}
### 使用ContentProvider

ContentProvider是Android中提供的专门用于不同应用间进行数据共享的方式。

关于ContentProvider的基本使用，包括建立一个ContentProvider和查询其他应用的ContentProvider，你在网上可以轻易地检索到。

在这里，笔者只讨论几个需要注意的店

* 运行线程问题：ContentProvider的onCreate方法运行在主线程中，其他插入、查询、更新、删除等方法运行在Binder线程池中。

* update、insert和delete方法会引起数据源的改变，这个时候我们需要通过ContentProvider的notifyChange方法来通知外界当前ContentProvider方法中的数据发生了变化。

外界想要了解，可以通过ContentResolver的registerContentObserver方法来注册观察者

* 增删查改四大方法都是存在多线程并发访问的，内部需要做好线程同步

### 使用Socket

Socket也被称为“套接字”，是网络通信中的概念，是网络通信中的概念，它分为流式套接字和用户数据报套接字。分别应用于TCP和UDP协议

TCP是面向连接的协议，提供稳定的双向通信，需要”三次握手“才能完成，为了提供稳定的传输，其本身提供了超时重传机制。UDP是无连接的，提供不稳定的单向通信功能。

## Binder连接池

Binder连接池是一种模式，这种模式的核心理念是，一个Service对应有多个Binder。在这种模式下，整个工作机制是，每个业务模块创建自己的AIDL接口并实现这个接口，然后向服务端提供自己唯一标识和其对应的Binder对象。服务端要提供一个queryBinder接口，可以让客户端去查询这个Binder连接池里的其他Binder。

参考书籍：Android开发艺术探索

