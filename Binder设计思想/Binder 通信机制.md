## Binder |  通信机制

### Binder 是什么?

> Binder 就是 Android 中的血管。在 Android 中所使用的 Activity、Service 等组件都需要和 AMS(system server 进程) 通信，这种跨进程通信都是通过 Binder 来完成的。

- 机制：Binder 是一种进程间通信机制
- 驱动：Binder 是一种虚拟物理设备驱动
- 应用层：Binder 是一个能发起通信 Java 类
- Framework/Native: Binder 连接了 client、server、ServiceManager 和 Binder 驱动程序，形成一套 C/S 的通信架构

如下图所示(很重要)：是整个 Binder 的通信流程，本篇文章也将围绕下图进行讲解 Binder 的设计思想。

<img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696843635026-53a84ae9-861c-4d01-bb2b-a38101ffd3ba.png" alt="img" style="zoom: 67%;" />

Android 系统分成三层，最上层是application应用层，第二层是Framework层，第三层是native层。

1.  Android中的**应用层**和**系统服务层**不在同一个进程，**系统服务**在单独的进程中。 
2.  Android中不同应用属于**不同的进程**中 
3.  Android应用和系统服务service运行在不同进程中是为了安全、稳定以及内存管理的原因，但是应用和系统服务需要通信和分享数据。 
4.  安全性：每个进行都是独立运行的，可以保证应用层对系统层的隔离 
5.  稳定性：如果某个进行崩溃了不会导致其他进程崩溃 
6.  内存分配：如果某个进程以及不需要了可以从内存中移除，并且回收相应的内存 

虚拟机分配给各个进程的运行内存是有限制的，`LMK(应用位于后台时的内存回收机制)`也会优先回收对系统资源的占用多的进程。

-  突破进程内存限制，如图库占用内存过多； 
-  功能稳定性：独立的通信进程保持长连接的稳定性；例如推送 有效的避免消息的丢失 
-  规避系统内存泄漏：独立的webview进程阻隔内存泄漏导致的问题； 
-  隔离风险：对于不稳定的功能放入独立进程，避免导致主进程崩溃； 

Binder 非常重要和很多原理有关：

1. 系统中的各个进程是如何通信的？
2. Android 系统启动过程
3. AMS、PMS 的原理
4. 四大组件的原理，比如 Activity 是如何启动的？
5. 插件化原理
6. 系统服务的 Client 端和 Server 端是如何通信的？

Binder 的机制主要分为以下几个部分，也是主要掌握的核心原理，如下图所示：

![img](https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1694310856210-23edcb38-28c9-43f9-993e-3c6dec95ca3c.jpeg)

### Android 为什么使用 Binder

**性能方面**：主要影响因素是拷贝次数，管道、消息队列、Socket 的拷贝次数都是两次，性能不是很好，共享内存不需要拷贝，性能最好，Binder 的拷贝次数为 1 次，性能仅次于内存拷贝。

**稳定性方面**：Binder 是基于 C/S 架构的，这个架构通常采用两层结构，在技术上已经很成熟了，稳定性是没有问题的。共享内存没有分层，难以控制，并发同步访问临界资源时，可能还会产生死锁，从稳定性的角度讲，Binder 是优于共享内存的。

**安全方面**：传统的 IPC 接收方无法获得对方可靠的进程用户 ID/进程 ID(UID/PID),无法鉴别对方身份。Android 为每个安装好的 APP 分配了自己的 UID，通过进程的 UID 来鉴别进程身份。另外 Android 系统中的 Server 端会判断 UID/PID 是否满足访问权限，而对外只暴露 Client 端，加强了系统的安全性。

**语言方面**：Linux 是基于 C 语言，C 语言是面向过程的，Android 应用层和 Java Framework 是基于 Java 语言，Java 语言是面向对象的。Binder 本身符合面向对象的思想，因此作为 Android 的通信机制更合适不过。

### IPC 机制

> IPC 全称：inter-Process Communication 进程间通信，指两个进程间进程数据交换的过程。

#### Linux 中的 IPC 机制种类

- **管道 Pipe**：主要思想是在内存中创建一个共享文件，从而使通信双方利用这个共享文件来传递信息。这个共享文件不属于文件系统只存在于内存中。(半双工通信方式)
  
  - 单工通信：只能在正或者反方向通信
  
  - 全双工通信：正反两个方向都可以进行通信 任何时候通信的两端都可以发送和接收数据

  - 半双工通信: 可以在两个方向上传输，但是某一时刻只能在一个方向上进行通信。
  
    <img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696922150367-dadaba64-9337-49cf-a0c6-824b809f7f2c.png" alt="img" style="zoom: 50%;" />
  
- **信号 Sinal**： 信号是软件层次上对中断机制的一种模拟，是一种异步通信方式，进程不必通过任何操作来等待信号的到达。信号可以在用户空间进程和内核之间直接交互，内核可以利用信号来通知用户空间的进程发生了哪些系统事件。信号不适用于信息交换，比较适用于进程中断控制。

- **信号量 Semophore**：信号量是一个**计数器**，用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某个进程正在访问共享资源时，其他进程也访问该资源。主要作为进程间以及同一进程内不同线程之间的同步手段。

- **消息队列 Message**：消息队列是消息链表，具有特定的格式，存放在内存中并由消息队列标识符标识，并且允许一个或多个进程向它写入与读取消息。信息会复制两次，因此对于频繁或者信息量大的通信不宜使用消息队列。

  <img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696922104631-b556b1b3-ac54-419f-93fa-cbf8651a6493.png" alt="img" style="zoom:50%;" />

- **共享内存 Share Memory**：多个进程可以直接读写的一块内存空间，是针对其他通信机制运行效率较低而设计的，为了在多个进程间交换信息，内核专门留出了一块内存区，可以由需要访问的进程将其映射到自己的私有地址空间。进程就可以直接读写一块内存而不需要进行数据的拷贝，从而大大的提高效率。**但是没有同步控制，多个进程同时访问，信息安全没有保障**。

- **套接字 Socket**：(全双工通信方式) 更为基础的进程间通信，套接字可用于不同机器间的进程间通信。

  <img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696922076976-1ac5ef38-2da7-4cf4-be70-b885f0457632.png" alt="img" style="zoom:50%;" />

#### Android 的 IPC 机制种类

Android 系统基于 Linux 内核的，在 Linux 内核基础上，又扩展出了一些 IPC 机制，Android 系统除了支持套接字，还支持序列化、Messenger、AIDL、Bundle、文件共享、ContentProvider、Binder 等。

- **序列化**：序列化是指 `Serializable/Parcelable`，`Serializable` 是 Java 提供的一个序列化接口，是一个空接口，为对象提供标准的序列化和反序列化操作。`Parcelable` 是 Android 中的序列化方式，更适合在 Android 平台上使用，用起来比较麻烦，但是效率很高。
- **Messenger**: `Messenger` 在 Android 应用开发中的使用频率不高，可以在不同进程中传递 `Message` 对象，在 `Message` 中加入传递的数据就可以在进程间进行数据传递了,`Messenger` 是一种轻量级的 IPC 方案并对 `AIDL` 进行了封装。
- **AIDL**：全名 Android interface definition Language,即 Android 接口定义语言。Messenger 是以串行的方式来处理客户端发来的消息，如果有大量的消息发到服务端，服务端仍然一个一个的处理，再响应客户端显然是不合适的。
- **Bundle**：Bundle 实现了 Parcelable 接口，所以它可以方便的在不同的进程间传输。Activity、Service、Receiver 都是在 Intent 中通过 Bundle 来进行数据传递。
- **文件共享**：两个进程通过读写同一个文件来进行数据共享，共享的文件可以是文本、XML、JOSN。文件共享适用于对数据同步要求不高的进程间通信。
- **ContentProvider**: `ContentProvider` 为存储和获取数据提供统一的接口，它可以在不同的应用程序之间共享数据，本身就是适合进程间通信的。`ContentProvider` 底层实现也是 `Binder`，但是使用起来比 AIDL 要容易许多。

### IPC 通信的原理

- 内核空间 Kernel Space (通常 32 位划分 1G 空间)：进程间数据共享
- 用户空间 User Space (通常 32 位划分为 3G 空间)：用户空间(用户程序)的数据是不共享的
- 进程隔离：一个进程不能直接操作和访问另一个进程，即使用户程序崩溃了 也不会影响其他进程和内核空间
- 系统调用：用户空间要访问内核空间就需要借助系统调用，系统调用是用户空间访问内核空间的唯一方式，所有的访问都是在内核空间控制下进行的,避免用户程序越权对系统资源访问。
- `copy_from_user`: 将用户空间数据拷贝到内核空间。
- `copy_to_user`: 将内核空间的数据拷贝到用户空间。

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1694160275244-e2536d30-37f4-45cd-9d27-232fe1b0a879.jpeg" alt="img" style="zoom:50%;" />

#### 内存映射

应用程序不能直接操作设备硬件地址，操作系统提供了一种机制叫做**内存映射(mmap)**，将设备地址映射到进程虚拟内存区。

假设不采用内存映射的情况，如下图所示：

用户程序想要访问文件，就需要两次拷贝

1. 读取文件请求到内核空间
2. 内核空间判断页缓存中是否有缓存，如果有直接返回；如果没有就去磁盘查找并**拷贝文件**到页缓存中。
3. 再将**文件拷贝**到用户空间

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1694307230872-105f0743-1949-47b0-81c6-172144801fb2.jpeg" alt="img" style="zoom:50%;" />



如果使用内存映射如何做的呢？如下图所示

通过 mmap 函数将硬盘文件映射到了内核空间的虚拟内存区域,建立映射关系后，用户对磁盘文件的修改可以直接反映到内核空间。只需要一次拷贝即可。

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1694307778319-84efd08b-36fc-4b76-a5a7-4dd3f63de5f9.jpeg" alt="img" style="zoom:50%;" />



#### Linux IPC 通信原理

1. 内核程序，在内核空间分配内存并开辟了一块内核的缓存区
2. 发送进程，通过 `copy_from_user` 将数据拷贝到内核空间的缓存区中
3. 接收进程，在接收数据时在自己的用户空间开辟一块内存缓存区
4. 内核程序，调用 `copy_to_user` 在内核缓存区将数据拷贝到接收进程

一次数据传递，需要两次的数据拷贝；接收数据的缓存区是接收进程提供，但是接收进程不知道多大的空间存放多大的数据，尽可能开辟尽可能大的空间，或者通过 API 接收消息头来获取消息体的大小来确定空间，浪费了空间和时间。

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1694309298144-ca50b81b-3d20-46b8-951a-220388a50bea.jpeg" alt="img" style="zoom:50%;" />



### Binder 的通信模型

> 内存划分：用户空间和内核空间，用户空间是用户程序代码运行的地方，内核空间是内核代码运行的地方，它们是隔离的，即使用户程序崩溃也不会影响内核。

`Binder` 的基本通信模型如下所示：单纯的以技术术语来讲，难以理解通过一个通俗易懂的例子来讲解 `Binder` 的通信模型运行的过程。

`Client` 要和 `Server` 通信，就需要拿到 `Server` 的 `Binder`，`Server` 端的 `Binder` 首先需要注册到 `ServiceManager`,`Client` 通过 `ServiceManager` 获取到 `Server` 端的 `Binder`，然后通过 `ioctl` 进行通信。

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1696753591819-f3e9de31-23b9-47d8-bcd7-59a779245a0b.jpeg" alt="img" style="zoom:50%;" />



只是看图和一些专业术语，很难理解Binder的通信机制，可能看的一脸懵逼，这里以网络通信的机制来展开 `Binder` 的通信模型机制，如下图所示：

个人电脑访问某个网址：www.baidu.com, 个人电脑相当于Client端，服务器相当于Server端

1. 首先个人电脑，访问服务器 www.baidu.com 的请求，转交给路由器，而不是直接请求服务器
2. 路由器通过 `DNS` 服务器，查到域名对应的 `IP` 地址，`DNS` 服务器维护了一个注册表(`域名:IP`)
3. 应用服务器，发布的时候会购买域名，对应的一个 `IP` 并注册到 `DNS` 服务器中
4. 路由器拿到 `IP` 地址返回给个人电脑
5. 个人电脑通过 `IP` 地址再通过路由器，找到 `IP` 对应的服务器地址，应用服务器将结果返回给路由器
6. 路由器将返回的结果，给个人电脑

可以看出，DNS 在网络通信的过程中起到了至关重要的作用， 一个网络请求是通过 DNS 来完成的

![img](https://cdn.nlark.com/yuque/0/2023/png/375694/1696750550595-30e770db-ef45-4b1b-a7e7-b3b78b2df572.png)

反过来再看 Binder 的通信模型：ServiceManager在Binder的通信模型中是至关重要的

- `ServiceManager` 其实就是等价于 `DNS`它存储了访问的IP地址(Binder)
- `ServiceManager` 存储方式类似`key:value`,value 就是Binder
- *`Binder 驱动`*可以理解为路由器，Client和Server通信都是通过`Binder驱动(路由器)`完成的

> 其实网络的通信机制和Binder通信机制类似，从现实中的访问网络例子可以更彻底的理解Binder的通信机制，看穿本质而不是死记硬背

再通过一个更直观，开发中常见的例子：`ActivityA` 启动 `ActivityB`,`Activity` 的启动是通过 `AMS` 来完成的，而 `AMS` 和`应用程序`不在一个进程中，它们之间的通信就是通过 `Binder` 来进行的，如下图所示：

1. 当手机开机启动的时候会创建 `init` 进程，`init进程`会 fork`zygote进程`，`zygote` 会 fork `SystemServer进程`，`SystemServer` 会创建和启动系统服务例如 `AMS` 然后注册到 `ServiceManager`中。如果你还不清楚 init/zygote/systemserver 等进程,会在后续详细讲解。
2. `ActivityA` 要启动 `ActivityB` 就需从 `ServiceManager`中拿到 `AMS` 的 `binder`。
3. 通过 `binder` 和 `AMS` 进行通信，启动 `ActivityB`。

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1696922919399-a0501c0f-23b6-44c9-af85-7a47a8983b94.jpeg" alt="img" style="zoom:50%;" />

### Binder 如何管理?

如下图所示，`SystemServer` 进程中的 `SystemServiceManager` 管理 `SystemService`。`ServiceManager` 管理 `binder` 注册表。

<img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696753594899-85e052e3-72f0-43fe-855d-c1a64bc5d312.png" alt="img" style="zoom: 80%;" />

`SystemServer` 进程是由 `zygote` 进程 `fork` 而来的，那么系统服务是如何将对应的 `binder` 注册到 `ServiceManager` 中呢？这里就需要深入源码中:

`framework/base/services/java/com/android/server/SystemServer.java`的 `run()` 方法中启动了系统服务,在 SystemServer 启动时就会调用 `run()` 方法：

```java
startBootstrapServices(t);
startCoreServices(t);
startOtherServices(t);
```

最终调用了`SystemServiceManager.startService`方法启动服务,调用了 `service` 的 `onStart()` 方法:

```java
public void startService(@NonNull final SystemService service) {
	// Register it.
	mServices.add(service);
	service.onStart();
}
```

以 `ATMS` 服务为例：`AMTS` 中的 `Lifecycle` 内部类继承了 `SystemServcie`

![img](https://cdn.nlark.com/yuque/0/2023/png/375694/1696757579497-c8326f09-7508-455d-a9fc-797fe39a4412.png)

```java
public static final class Lifecycle extends SystemService {
    private final ActivityTaskManagerService mService;

    public Lifecycle(Context context) {
        super(context);
        mService = new ActivityTaskManagerService(context);
    }

    @Override
    public void onStart() {
        publishBinderService(Context.ACTIVITY_TASK_SERVICE, mService);
        mService.start();
    }
}
```

在上述代码中可以看到在 `onStart` 方法中调用了一个方法 `publishBinderServcie()`这个方法就是将 `ATMS` 服务注册到 `ServiceManager` 中去。

name: `Context.ACTIVITY_TASK_SERVICE （"activity_task"）`
value: `mService`

value` 为什么是 `mService呢？因为继承了`IActivityTaskManager.Stub`就是 `binder`

```java

protected final void publishBinderService(String name, IBinder service,
                                          boolean allowIsolated, int dumpPriority) {
    ServiceManager.addService(name, service, allowIsolated, dumpPriority);
}

```

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1696758285004-93cd43c1-29b0-4c95-94e7-b3804e757bad.jpeg" alt="img" style="zoom:50%;" />

在应用开发中类似的还有 `MediaPlayer` 框架也是通过 `Binder` 进行进程间的通信：可见 `ServiceManager` 在 `Binder` 通信中起到了至关重要的作用。

<img src="https://cdn.nlark.com/yuque/0/2023/png/375694/1696922367947-2bed7719-d454-4a64-9233-fc829078ff9c.png" alt="img" style="zoom:50%;" />



那么思考另一个问题，如何通过 `ServiceManager` 获取 `binder` 呢？

`ServiceManager` 是在一个单独的进程中，通过Init进程解析rc文件启动ServiceManager进程， 也是一个跨进程的，怎么跨进程调度呢？

再来看下图的网络通信模型，DNS 服务器的地址是开放的：`8.8.8.8`，那么路由器就可以直接通过访问开放的 IP 地址访问 DNS 服务器了。

![img](https://cdn.nlark.com/yuque/0/2023/png/375694/1696758703996-ae9c0ca7-cd4e-4054-8c6e-3d5f80a266d1.png)

那么 `ServiceManager` 也是相似的原理，在 Android 中是通过 `handle`,每一个 `binder` 都会有一个 `handle`，那么 `handle` 为 **0** 的就是 `ServiceManager`, `Client` 端 可以直接通过 `handle` 为 **0** 获取到 `ServcieManager` 的 `binder` 进行通信，然后获取到目标 `Server` 端的 `binder`。

![img](https://cdn.nlark.com/yuque/0/2023/png/375694/1696759091021-d9bc9414-a101-404f-874c-3b3fd239f4bd.png)

每一个 binder 通信都需要通过 Binder 驱动来完成，其实 Binder 驱动就是一个协议,下面来看看讲解 binder 驱动做了什么?

### Binder 通信原理

Binder 是基于**内存映射(mmap)**实现的，内存映射通常是用在有物理介质的文件系统上的，Binder 没有物理介质，它使用内存映射是用于实现跨进程传递数据。

<img src="https://cdn.nlark.com/yuque/0/2023/jpeg/375694/1696841638261-ade2f0a5-4a45-4b5f-b6ed-13f23c26f53d.jpeg" alt="img" style="zoom:50%;" />



### Binder 是如何创建的?

上述讲解清楚了 `Binder` 的通信机制，那么 `Binder` 是什么时候创建的呢？众所周知 `zygote` 进程孵化 `app` 进程和` system server` 进程，那么 `zygote` 进程有自己的 `binder` 吗？要搞清楚这一块就需要去源码中查看, **这里先不深入讲解，只了解流程。**

其实在进程创建的时候，就准备了 Binder 驱动. Binder 的创建其实就是 Binder 驱动初始化。

PS：这里需要你先要了解 Android 的系统启动流程，否则会看的云里雾里

#### ServiceManager 进程

如下代码注释：ServiceManager 进程在初始化执行了 main 函数，而主要做的事情就是创建 binder 驱动

```cpp
int main(int argc, char** argv) {
	//根据上面的rc文件，argc == 1,argv[0]=="system/bin/servicemanager"
	if (argc > 2) {
		LOG(FATAL) << "usage: " << argv[0] << " [binder driver]";
	}
	//此时使用binder驱动为：/dev/binder
	const char* driver = argc == 2 ? argv[1] : "/dev/binder";
	//1. 初始化binder驱动
	sp<ProcessState> ps = ProcessState::initWithDriver(driver);
	ps->setThreadPoolMaxThreadCount(0);
	ps->setCallRestriction(ProcessState==CallRestriction==FATAL_IF_NOT_ONEWAY);
	//实例化ServiceManager
	sp<ServiceManager> manager = new ServiceManager(std::make_unique<Access>());
	//将自身作为服务添加到 ServiceManager
	if (!manager->addService("manager", manager, false /*allowIsolated*/, IServiceManager::DUMP_FLAG_PRIORITY_DEFAULT).isOk()) {
		LOG(ERROR) << "Could not self register servicemanager";
	}
	//设置服务端BBinder对象
	IPCThreadState::self()->setTheContextObject(manager);
	//设置成为binder驱动的context manager,成为上下文管理者
	ps->becomeContextManager(nullptr, nullptr);
	//通过Looper epoll机制处理binder事务
	sp<Looper> looper = Looper::prepare(false /*allowNonCallbacks*/);

	BinderCallback::setupTo(looper);
	ClientCallbackCallback::setupTo(looper, manager);

	while(true) {
		looper->pollAll(-1);
	}

	// should not be reached
	return EXIT_FAILURE;
}
```

关键函数就是 `initWithDriver`执行如下，主要看 `open_driver` 函数，打开 `binder` 驱动：

```cpp
ProcessState::ProcessState(const char *driver)
    : mDriverName(String8(driver))
    , mDriverFD(open_driver(driver))
    , mVMStart(MAP_FAILED)
    , mThreadCountLock(PTHREAD_MUTEX_INITIALIZER)
    , mThreadCountDecrement(PTHREAD_COND_INITIALIZER)
    , mExecutingThreadsCount(0)
    , mMaxThreads(DEFAULT_MAX_BINDER_THREADS)
    , mStarvationStartTimeMs(0)
    , mBinderContextCheckFunc(nullptr)
    , mBinderContextUserData(nullptr)
    , mThreadPoolStarted(false)
    , mThreadPoolSeq(1)
    , mCallRestriction(CallRestriction::NONE)
```

`open_driver` 看看做了什么,就是打开 binder 驱动的代码，这里先不去深入研究，通过上述的代码可以看出在创建 `ServiceManager` 进程的时候就打开了 binder 驱动。

```cpp
static int open_driver()
{
    //打开binder驱动
    int fd = open("/dev/hwbinder", O_RDWR | O_CLOEXEC);
    if (fd >= 0) {
        int vers = 0;
        //验证binder版本
        status_t result = ioctl(fd, BINDER_VERSION, &vers);
        if (result == -1) {
            ALOGE("Binder ioctl to obtain version failed: %s", strerror(errno));
            close(fd);
            fd = -1;
        }
        if (result != 0 || vers != BINDER_CURRENT_PROTOCOL_VERSION) {
            ALOGE("Binder driver protocol(%d) does not match user space protocol(%d)!", vers, BINDER_CURRENT_PROTOCOL_VERSION);
            close(fd);
            fd = -1;
        }
        //设置binder最大线程数
        size_t maxThreads = DEFAULT_MAX_BINDER_THREADS;
        result = ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);
        if (result == -1) {
            ALOGE("Binder ioctl to set max threads failed: %s", strerror(errno));
        }
    } else {
        ALOGW("Opening '/dev/hwbinder' failed: %s\n", strerror(errno));
    }
    return fd;
}
```

#### App 进程

> 再来看看应用进程，应用进程肯定会用到 Binder，`zygote` fork 了 App 进程这里的流程不在复述，可以看之前的文章有详细讲解 Zygote 进程。创建 App 进程肯定会执行 `AppRuntime.onZygoteInit()` 方法

```java
public static final Runnable zygoteInit(int targetSdkVersion, long[] disabledCompatChanges,String[] argv, ClassLoader classLoader) {
    ......
    RuntimeInit.commonInit();//初始化运行环境
    ZygoteInit.nativeZygoteInit();//启动Binder，方法在AndroidRuntime.cpp中注册
    //通过反射创建程序入口函数的Method对象，并返回Runnable函数，执行ActivityThread.main()
    return RuntimeInit.applicationInit(targetSdkVersion, disabledCompatChanges, argv,
                                       classLoader);
}
```

`nativeZygoteInit` 函数执行代码：

```cpp
    virtual void onZygoteInit()
    {
        sp<ProcessState> proc = ProcessState::self();
        ALOGV("App process: starting thread pool.\n");
        proc->startThreadPool();
    }
```

同样也是调用了 `ProcessState`，需要注意的是 `ProcessState` 和 `ServiceManager` 的 `ProcessState` 不是同一个 `cpp` 文件。App 进程调用的 ProcessState 路径`system/libhwbinder/ProcessState.cpp`,同样也会调用 `open_driver` 函数打开 binder 驱动。

为什么 App 进程在初始化的时候第一事件初始化了 Binder 驱动？

> App 进程创建就需要面临启动 Activity，而启动 Activity 就需要 AMS 来启动，就需要 Binder 进行通信，所以要第一时间初始化 Binder 驱动。

Zygote 进程有 Binder 吗？答案是**没有**

> 因为 Zygote 进程是通过 `socket` 来进行通信的，原由是因为 fork 孵化进程，不允许有多线程存在的，有可能会产生死锁，而 Binder 的底层通信支持多线程，例如 AMS 作为 server 端，那么会有多个 Client 端和 Server 端通信，那么 Binder 会在每一次通信创建一个独立的线程进行通信。所以 Zygote 进程采用了 Socket 进行跨进程通信。

> 本篇文章讲解了Binder的Native流程及通信机制，下一篇讲解应用层Binder在应用开发中如何使用Binder进行进程间通信。

更详细的Binder的通信流程如下图所示，会在后续的文章中详细的讲解：

![binder_flow](./Binder%20%E9%80%9A%E4%BF%A1%E6%9C%BA%E5%88%B6.assets/binder_flow.jpeg)