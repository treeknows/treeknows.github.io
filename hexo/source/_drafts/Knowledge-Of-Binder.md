---
title: Knowledge-Of-Binder
tags:

---

s



Linux提供了多种IPC方式，如管道、消息队列、共享内存、Socket等，为什么Android还要提供Binder来进行IPC通信呢

- 性能

- 稳定性
- 安全性



Linux进程间通信的基本概念：

- 进程隔离
- 进程空间：用户空间/内核空间
- 系统调用：用户态/内核态

Linux传统IPC通信的原理：Process A在用户空间申请内存，通过系统调用进入**内核态**，在内核空间分配内存，开辟一块内核缓冲区。调用`copyfromuser()`函数将用户空间的内存数据拷贝到内核缓冲区。同样的Process B在用户空间开辟一块内存缓冲区，通过系统调用进入**内核态**，内核程序调用`copytouser()`函数将数从内核缓冲区拷贝到自己的用户空间内存缓冲区。这样就完成了一次进程间的数据传输。

缺点有二

1. 性能低下，传输一次数据需要经过两次内存拷贝：A内存缓冲区-->内核缓冲区-->B内存缓冲区
2. 接收数据的缓冲区由接收进程开辟，接收进程B不知道数据量有多大，只能尽可能的开辟大的内存空间或者先调用API接收下消息头来获取数据大小，这两种做法不是浪费空间就是浪费时间



Binder

如上所述，跨进程通信是需要内核空间做支持的。传统的IPC如管道、Socket都是内核的一部分，因此通过内核支持实现进程间通信是没问题的。但是Binder并不是Linux系统的一部分，只能通过Linux的**动态内核可加载模块**（Loadable Kernel Module，LKM）机制来实现。对于Linux来说，模块是具有独立功能的程序，它可以被单独编译，但是不能独立运行。它在运行时被链接到内核作为内核的一部分运行。通过这种方式，Android系统就可以通过动态添加一个内核模块运行在内核空间，Binder就是这样的一个模块，用户可以通过Binder作为桥梁来实现进程间通信。

> 在Android系统中，这个运行在内核空间，负责各个用户进程通过Binder实现通信的内核模块就叫做**Binder驱动**（Binder Driver）









Binder在Android中的应用

Binder分为实名Binder和匿名Binder

实名Binder即通过`ServiceManager.addService`中注册过的Binder Server，如常见的AMS、ATMS等，亦或者自己实现的一些系统服务。

实名Binder在使用时，可以通过`ServiceManager.getService()`方法获取Binder的代理，通过该代理调用接口方法，之后再通过Binder驱动到Server端执行方法，返回结果。但是对于一般的系统服务如AMS来说，Android提供了ActivityManager.java，其实例可以通过`Context.getSystemService(Context.ACTIVITY_SERVICE)`来获取。`ActivityManager`其实只是一个壳子，调用其方法其实就是在调用AMS代理类的方法。

```
// 调用示例
activityManager.getRunninngAppProcesses();

// 方法的实现如下
public List<RunningAppProcessInfo> getRunningAppProcesses() {
    try {
        return getService().getRunningAppProcesses();        
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}

// 可以看到调用的其实是getService()的同名方法
public static IActivityManager getService() {
    return IActivityManagerSingleton.get();
}

// 实际获取了IActivityManager的proxy对象
private static final Singleton<IActivityManager> IActivityManagerSingleton =
        new Singleton<IActivityManager>() {
            @Override
            protected IActivityManager create() {
                final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                final IActivityManager am = IActivityManager.Stub.asInterface(b);
                return am;
            }
        };

```

匿名Binder就类似于两个应用间通过AIDL，以`context.bindService()`的方式进行通信的一种方式。

在Client App中，执行`bindService`方法时，该方法的一个入参类型为`ServiceConnection`，而我们一般初始化`ServiceConnection`对象时，会重写其`onServiceConnected`方法，该方法会返回一个`IBinder`类型的参数，可以通过该参数获得自定义AIDL的proxy对象

```
// 示例代码DEMO
    //声明IBookManager类型的对象
    private IBookManager mIBookManager;

    //创建连接
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //关键代码1，获取IBookManager类型的对象
            mIBookManager = IBookManager.Stub.asInterface(service);

			//关键代码2，打印出mIBookManager的真实类型
            if (mIBookManager instanceof IBookManager.Stub) {
                Log.e("znh", "mIBookManager instanceof IBookManager.Stub");
            }
            Log.e("znh", "mIBookManager.getClass().getName():" + mIBookManager.getClass().getName());
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

```





