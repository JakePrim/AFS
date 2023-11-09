# ServiceManager的Binder机制

> Java binder 是借助Native Binder来进行工作的，ServiceManager用于管理系统中的服务，ServiceManager的机制就是Native Binder的核心机制。(PS: 建议在学习Native层代码之前 学习C/C++和JNI相关的知识，否则底层代码看不懂)

回顾前几篇文章，在《Binder通信机制》如下图的通信流程：可以看到在Native层中`ServiceManager`就是Native Binder的机制。

<img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696843635026-53a84ae9-861c-4d01-bb2b-a38101ffd3ba.png" alt="img" style="zoom: 67%;" />

本文中讲解源码的路径：

```
framework/native/libs/binder/
  - ProcessState.cpp
  - BpBinder.cpp
  - Binder.cpp
  - IServiceManager.cpp

framework/native/include/binder/
  - IServiceManager.h
  - IInterface.h
  
frameworks/av/media/mediaserver/
  - main_mediaserver.cpp	
```

以`MediaPlayerService`系统多媒体服务为例，展开讲解`ServiceManager`的Binder机制是如何实现的。`MediaPlayerService`系统服务是由`MediaServer`进程提供的，是一个可执行的程序，在系统启动的时候`MediaServer`进程就会启动。

`MediaPlayerService`服务的入口函数如下：

代码路径：`frameworks/av/media/mediaserver/main_mediaserver.cpp`

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
    gProcess = new ProcessState("/dev/binder");
    return gProcess;
}
```

下面来看ProcessState的构造函数如下：

- 注释1：打开/dev/binder 设备驱动
- 注释2：调用了mmap函数，实现了内存映射，会在内核的虚拟地址空间中申请一块与用户虚拟内存相同大小的内存，然后申请物理内存，将同一块物理内存分别映射到内存虚拟地址空间和用户虚拟内存空间实现内核虚拟地址空间和用户虚拟内存空间的数据同步操作,通过mmap为binder分配一块虚拟地址空间，达到内存映射的目的。

```cpp
ProcessState::ProcessState(const char *driver)
    : mDriverName(String8(driver))
        , mDriverFD(open_driver(driver))//1
        , mVMStart(MAP_FAILED)
        , ....
        , mThreadPoolSeq(1)
    {
        if (mDriverFD >= 0) {
            // mmap the binder, providing a chunk of virtual address space to receive transactions.
            mVMStart = mmap(nullptr, BINDER_VM_SIZE, PROT_READ, MAP_PRIVATE | MAP_NORESERVE, mDriverFD, 0);//2
            if (mVMStart == MAP_FAILED) {
                // *sigh*
                ALOGE("Using %s failed: unable to mmap transaction memory.\n", mDriverName.c_str());
                close(mDriverFD);
                mDriverFD = -1;
                mDriverName.clear();
            }
        }
    }
```

open_driver 函数做了什么呢？

- 注释1：调用了open函数，用于打开 dev/binder设备并返回文件操作符FD，这样就可以操作内核的binder驱动了。
- 注释2：ioctl 函数和**binder设备进行参数的传递**，这里的作用是用于设置binder支持 最大支持线程数maxThreads->15
- 最终返回的值就是fd文件操作符。

```cpp
static int open_driver(const char *driver)
{
    int fd = open(driver, O_RDWR | O_CLOEXEC);//1
    if (fd >= 0) {
        .....
        size_t maxThreads = DEFAULT_MAX_BINDER_THREADS;
        result = ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);//2
        if (result == -1) {
            ALOGE("Binder ioctl to set max threads failed: %s", strerror(errno));
        }
    } else {
        ALOGW("Opening '%s' failed: %s\n", driver, strerror(errno));
    }
    return fd;
}
```

ProcessState 主要做了几件事：

- 打开/dev/binder设备并设定Binder的最大的支持线程数。
- 通过mmap为binder分配一块虚拟地址空间，达到内存映射的目的。

### defaultServiceManager

> 下面来看defaultServiceManger函数做了什么事？

代码如下：

使用了单例模式，看注释1处代码`ProcessState::self()->getContextObject(nullptr)` ，`ProcessState` 在上述的讲解中`self()`函数也是使用了单例模式获取`ProcessState` 的实例。

代码路径：`frameworks/native/libs/binder/IServiceManager.cpp`

```cpp
sp<IServiceManager> defaultServiceManager()
{
    std::call_once(gSmOnce, []() {
        sp<AidlServiceManager> sm = nullptr;
        while (sm == nullptr) {
            sm = interface_cast<AidlServiceManager>(ProcessState::self()->getContextObject(nullptr));//1
            if (sm == nullptr) {
                ALOGE("Waiting 1s on context object on %s.", ProcessState::self()->getDriverName().c_str());
                sleep(1);
            }
        }

        gDefaultServiceManager = new ServiceManagerShim(sm);
    });

    return gDefaultServiceManager;
}
```

再来看getContextObject函数做了什么事情：

- 注释1：调用了`getStrongProxyForHandle`并且把`handle=0`做为参数进行传递，在第一篇文章中《Binder的通信机制》，`ServiceManager` 是在单独的进程中的，也是通过`binder`进行通信的，并且每个`binder`都会有一个`handle`,而`ServiceManager`的binder的handle固定就是`0` handle是一个资源标识。
- 注释2：`lookupHandleLocked` 函数是用来查询对应资源标识的资源,也就是`handle_entry`是否存在`binder`，如果不存在就会调用注释3出的`create`函数
- 注释3：用于新建`BpBinder`,赋值给`handle_entry.binder` 最终返回的值就是`BpBinder`

```cpp
sp<IBinder> ProcessState::getContextObject(const sp<IBinder>& /*caller*/)
{
    sp<IBinder> context = getStrongProxyForHandle(0);//1
    ...
        return context;
}
sp<IBinder> ProcessState::getStrongProxyForHandle(int32_t handle)
{
    sp<IBinder> result;
    AutoMutex _l(mLock);
    handle_entry* e = lookupHandleLocked(handle);//2
    if (e != nullptr) {
        IBinder* b = e->binder;
        if (b == nullptr || !e->refs->attemptIncWeak(this)) {
            ....
            b = BpBinder::create(handle);//3
            e->binder = b;
            if (b) e->refs = b->getWeakRefs();
            result = b;
        } else {
            result.force_set(b);
            e->refs->decWeak(this);
        }
    }
    return result;
}
```

那么问题来了什么是`BpBinder`呢？`BpBinder`和`BBinder` 之间有什么关系呢？

都继承了`IBinder`,`BpBinder`是Client端和Server端交互的代理类，`BpBinder`会通过handle找到对应的`BBinder`

![image-20231031172800124](./ServiceManager%E7%9A%84Binder%E6%9C%BA%E5%88%B6.assets/image-20231031172800124.png)

> ProcessState::getContextObject 就是拿到BpBinder用于和Server端进行交互

回头再来看`defaultServiceManager`中的`interface_cast`函数

如下代码：采用了C++中的模板实现

`frameworks/native/libs/binder/include/binder/IInterface.h`

```c++
template<typename INTERFACE>
inline sp<INTERFACE> interface_cast(const sp<IBinder>& obj)
{
    return INTERFACE::asInterface(obj);
}
```

在`defaultServiceManager`中传递的`INTERFACE`是`IServiceManager`

```c++
interface_cast<AidlServiceManager>(ProcessState::self()->getContextObject(nullptr))
using AidlServiceManager = android::os::IServiceManager;
```

下面来看IServiceManager的实现继承了IInterface：

```c++
class IServiceManager : public IInterface
{
public:
    //一些操作service函数
    virtual sp<IBinder>         getService( const String16& name) const = 0;
    virtual sp<IBinder>         checkService( const String16& name) const = 0;
    virtual status_t addService(const String16& name, const sp<IBinder>& service,
                                bool allowIsolated = false,
                                int dumpsysFlags = DUMP_FLAG_PRIORITY_DEFAULT) = 0;
    virtual Vector<String16> listServices(int dumpsysFlags = DUMP_FLAG_PRIORITY_ALL) = 0;
};  
```

对于asInterface()函数，通过搜索代码，你会发现根本找不到这个方法是在哪里定义这个函数的, 其实是通过模板函数来定义的，通过下面两个代码完成的：

 libbinder 中的 `ServiceManagerShim` 起到了一个中转的作用，把请求转给 out 下 aidl 自动生成的 `BpServiceManager`

- `BpServiceManager` 的实现挪到 out

原来是在 `libbinder#IServiceManager.cpp` 中手写实现，现在是 `aidl` 帮你实现。

![image-20231108090021855](./ServiceManager%E7%9A%84Binder%E6%9C%BA%E5%88%B6.assets/image-20231108090021855.png)

IServiceManager的代码目录：

![image-20231108091219816](./ServiceManager%E7%9A%84Binder%E6%9C%BA%E5%88%B6.assets/image-20231108091219816.png)

当然，该文件中同样自动实现了 BnServiceManager 类

```cpp
//位于IServiceManager.h文件
DECLARE_META_INTERFACE(ServiceManager)
//位于IServiceManager.cpp文件
IMPLEMENT_META_INTERFACE(ServiceManager,"android.os.IServiceManager")
```





- defaultServiceManager 等价于 `new BpServiceManager(new BpBinder(0));`

ProcessState::self()主要工作：

- 调用open()，打开/dev/binder驱动设备；
- 再利用mmap()，创建大小为`1M-8K`的内存地址空间；
- 设定当前进程最大的最大并发Binder线程个数为`16`。

BpServiceManager巧妙将通信层与业务层逻辑合为一体，

- 通过继承接口`IServiceManager`实现了接口中的业务逻辑函数；
- 通过成员变量`mRemote`= new BpBinder(0)进行Binder通信工作。
- BpBinder通过handler来指向所对应BBinder, 在整个Binder系统中`handle=0`代表ServiceManager所对应的BBinder。

