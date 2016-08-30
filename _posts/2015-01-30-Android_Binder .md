---
layout:     post
title:      "Android Binder"
subtitle:   ""
date:       2015-01-30 11:00:00
author:     "steven"
catalog: true
tags:
    - Android
---

android是基于Linux的系统，进程之间是相互隔离的，Binder就是开发出来用以解决进程之间通信的(IPC)。

实现进程之间的通信Linux有多重实现方式，由于android是手机操作系统，内存受限，需要合适的机制来保证对空闲进程的回收，还有系统安全问题，移动平台权限问题,进程终止通知的需求，况且android不支持 system V IPCs.所以设计开发了Binder来处理android系统进程之间的通信。


Binder是基于内存共享的方式实现进程之间的通信的，Binder Driver负责处理进程之间的调用命令，Binder Driver被放置于所有进程都能共享的区域-Kernel.Binder Driver让各进程使用内核空间，将进程中的地址和Kernel中的地址映射起来。

Binder是基于内存共享的方式实现进程之间的通信的，Binder Framework基于C/S建构实现，客户端需要调用远程服务的内容时， 会初始化一个连接，并等待服务器的返回，同时会阻塞住自己。Binder Framework在客户端这边实现了一个代理，而在服务端，通过线程池的方式来响应请求。

binder传递数据的格式
---
实现跨进程调用涉及到参数和命令的传递。格式：
target | Binder Driver Command | Cookie | Sender ID | Data

 target为目标Binder
 Cookie涵盖着一些内部信息
 Sender ID 包含了安全相关的信息，标识Binder.
 data包含着一些数据的数组，每个Entry是由相关的命令和参数组成。

Service Manager
----

Service Manager负责通过服务名称找到对应的服务地址，类似DNS。客户端不应该知道远程服务的调用地址，如果知道了会不安全。Binder需要将自己的名字和Binder Token交给Service Manager，客户端只需要知道服务的名字就可。
应用程序(Client) 首先向 Service Manager 发送请求对应Manager(如 ActivityService 、 WindowMananger) 的服务，Service Manager 查看已经注册在里面的服务的列表，找到相应的服务后，通过 Binder kernel 将其中的 Binder 对象返回给客户端，从而完成对服务的请求。

在 Binder Framework 中 Binder 充当了 Socket 的角色,对Client而言，Binder可以看成是通向Server的管道入口，要想和某个Server通信首先必须建立这个管道并获得管道入口.对Server而言，Binder可以看成Server提供的实现某个特定服务的访问接入点,Client通过这个'地址'向Server发送请求来使用该服务

Service Manager 首先就被创建，并被赋予了一个特殊的句柄，这个句柄就是 0 。其他 Server 进程都可以通过这个 0句柄 与 Service Manager 进行通信，在整个系统启动时，其他 Server 进程都向这个 0句柄进行注册，从而使得客户端进程在需要调用服务时，能够通过这个 Service Manager 查询到相应的服务进程。

Binder Framework实现使用了代理模式，首先定义一套相同的接口，服务端 和 客户端分别使用这套接口，服务端具体实现了这套接口的相应逻辑，而客户端也实现了这套接口，不过接口里面的具体实现是调用相应的远程服务接口，将函数参数打包，通过Binder向Server发送申请并等待返回值。

IBinder 与 Binder
----

Android系统将一个可远程操作的应用定义为 IBinder，在这个接口中定义了一个可远程调用对象应该具有的属性和方法，在代码中实际使用的Binder 也都是继承自 IBinder 对象。 IBinder 抽象了远程调用的接口，任何一个可远程调用的对象都应该实现这个接口


```java
public interface IBinder {
    ... ...

    // 查看 binder 对应的进程是否存活
    public boolean pingBinder();
    // 查看 binder 是否存活，需要注意的是，可能在返回的过程中，binder 不可用
    public  boolean isBinderAlive();

    /**
     * 执行一个对象的方法，
     *
     * @param 需要执行的命令
     * @param 传输的命令数据，这里一定不能为空
     * @param 目标 Binder 返回的结果，可能为空
     * @param 操作方式，0 等待 RPC 返回结果，1 单向的命令，最常见的就是 Intent.
     */
    public boolean transact(int code, Parcel data, Parcel reply, int flags) throws RemoteException;

    // 注册对Binder死亡通知的观察者，在其死亡后，会收到相应的通知
    // 这里先跳过，后续
    public void linkToDeath(DeathRecipient recipient, int flags) throws RemoteException;

    ... ...

}
```

transact方法会根据Uid 和 Pid （用户id和进程id）进行相应的校验，校验通过后，将相应的数据写入 writeTransactionData，其后在 waitForResponse 里面读取前面写入的值，并执行相应的方法，最后返回结果。
由于 IBinder 对象是一个高度抽象的结构，直接使用这个接口对于应用层的开发者而言学习成本太高，需要涉及到不少本地实现，因而 Android 实现了 Binder 作为 IBinder 的抽象类，提供了一些默认的本地实现，当开发者需要自定义实现的时候，只需要重写 Binder 中的 protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) 方法即可。

AIDL
----

AIDL(Android Interface Definition Language) 被Android 系统广泛使用，在许多地方都能看到相应的场景。是一种IDL 语言,用于生成可以在Android设备上两个进程之间进行进程间通信.

AIDL=Binder+开发者自定义接口

aidl文件中声明要调用的方法，aidl中支持的参数类型为：基本类型（int,long,char,boolean等）,String,CharSequence,List,Map，其他类型必须使用import导入,即便在该类和定义的包在同一个包中.自定义类型必须实现Parcelable接口.接口中的参数除了aidl支持的类型，其他类型必须标识其方向：到底是输入还是输出抑或两者兼之，用in，out或者inout来表示.Java原始类型默认的标记为in.

然后通过aidl的代码生成器，根据接口自动生成了aidl代码文件,继承Stub类实现接口调用的方法。
