# ServiceManager注册服务的过程

> 在前几篇文章中Java Binder 时候，提到了一些系统服务(如：AMS、PMS等)会注册到ServiceManager中，那么具体是如何注册的呢？本篇文章详细讲解。以`MediaPlayerService`服务的注册过程为例

先来看整体的流程，从函数的调用角度来看：

1. `addService`函数将数据打包发送给`BpBinder`来进行处理。
2. `BpBinder`新建了一个`IPCThreadState`对象，并将通信的任务交给`IPCThreadState`。
3. `IPCThredState`的`writeTransactionData`函数用于将命令协议和数据写入到`mOut`中。
4. `IPCThreadState`中的`waitForResponse`函数主要做了两件事，一件事是通过`ioctl`函数操作`mOut`和`mIn`来与Binder驱动进行数据交互，另一件事是处理各种命令协议。

MediaPlayerService的入口函数：

`frameworks/av/media/mediaserver/main_mediaserver.cpp`

```cpp
int main(int argc __unused, char **argv __unused)
{
    signal(SIGPIPE, SIG_IGN);
    //获取ProcessState实例
    sp<ProcessState> proc(ProcessState::self());//1 初始化binder驱动
    sp<IServiceManager> sm(defaultServiceManager());//2 获取ServiceManager
    ALOGI("ServiceManager: %p", sm.get());
    AIcu_initializeIcuOrDie();
    MediaPlayerService::instantiate();//3 注册MediaPlayerService服务
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

