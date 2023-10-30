# ServiceManager的Binder机制

> Java binder 是借助Native Binder来进行工作的，ServiceManager用于管理系统中的服务，ServiceManager的机制就是Native Binder的核心机制。

回顾前几篇文章，在《Binder通信机制》如下图的通信流程：可以看到在Native层中`ServiceManager`就是Native Binder的机制。

<img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696843635026-53a84ae9-861c-4d01-bb2b-a38101ffd3ba.png" alt="img" style="zoom: 67%;" />

以MediaPlayerService系统多媒体服务为例，展开讲解ServiceManager的Binder机制是如何实现的。我们知道MediaPlayerService系统服务是由MediaServer进程提供的，是一个可执行的程序，在系统启动的时候MediaServer进程就会启动。

MediaPlayerService服务的入口函数如下：

`frameworks/av/media/mediaserver/main_mediaserver.cpp`

```cpp
int main(int argc __unused, char **argv __unused)
{
    signal(SIGPIPE, SIG_IGN);
    //获取ProcessState实例
    sp<ProcessState> proc(ProcessState::self());//1. 打开dev/binder设备,并使用mmap函数为binder驱动分配虚拟地址空间用来接收数据。
    sp<IServiceManager> sm(defaultServiceManager());//2. 获取IServiceManager
    ALOGI("ServiceManager: %p", sm.get());
    AIcu_initializeIcuOrDie();
    MediaPlayerService::instantiate();//3. 注册MediaPlayerService服务
    ResourceManagerService::instantiate();
    registerExtensions();
    ::android::hardware::configureRpcThreadpool(16, false);
    //启动binder线程池
    ProcessState::self()->startThreadPool();
    //当前线程加入到线程池
    IPCThreadState::self()->joinThreadPool();
    ::android::hardware::joinRpcThreadpool();
}
```

### ProcessState

> 每个进程唯一的ProcessState，`self`函数采用了单例的设计模式，确保每一个进程只有一个ProcessState实例。

```cpp
sp<ProcessState> ProcessState::self()
{
    Mutex::Autolock _l(gProcessMutex);
    if (gProcess != nullptr) {
        return gProcess;
    }
    gProcess = new ProcessState(kDefaultDriver);
    return gProcess;
}
```



