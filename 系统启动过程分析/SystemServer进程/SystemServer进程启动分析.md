# SystemServer进程启动过程分析

> 在上一篇文章中，讲解了Zygote的启动原理，在启动Zygote进程的过程中，同时启动了System Server进程，本章主要详细讲解SystemServer进程的启动过程。

本篇文章需要知道Zygote进程的启动原理，建议先阅读上一篇：
[Framework | Zygote进程启动原理](https://www.yuque.com/jakeprim/android/mwgg3e3d2mkt2yf9?view=doc_embed)

> SystemServer进程主要作用是用来创建系统服务。本篇主要讲解SystemServer进程启动的详细过程及如何管理、启动系统服务。

### Zygote处理SystemServer进程
> `Zygote` 进行在`ZygoteInit.java`类中调用了`forkSystemServer`通过JNI调用`native fork SystemServer`进程。

在注释2处，返回了`Runnable`？ 为什么返回了一个`Runnable`实例呢？这里先思考 留下疑问 后面详解
```cpp
//1. 创建了ZygoteServer内部创建一个server端的socket，用于等待AMS请求socket来创建应用程序进程
zygoteServer = new ZygoteServer(isPrimaryZygote);
if (startSystemServer) {
    //2. fork systemServer进程
    Runnable r = forkSystemServer(abiList, zygoteSocketName, zygoteServer);
    // 调用SystemServer.main()方法
    if (r != null) {
        r.run();
        return;
    }
}
//3. 开始轮询，等待AMS请求
caller = zygoteServer.runSelectLoop(abiList);
```
下面看`forkSystemServer`如何启动`SystemServer`进程的，代码如下：
```cpp
/* Request to fork the system server process */
//1.创建一个子进程SystemServer
pid = Zygote.forkSystemServer(
    parsedArgs.mUid, parsedArgs.mGid,
    parsedArgs.mGids,
    parsedArgs.mRuntimeFlags,
    null,
    parsedArgs.mPermittedCapabilities,
    parsedArgs.mEffectiveCapabilities);
} catch (IllegalArgumentException ex) {
    throw new RuntimeException(ex);
    }

    //2. 当前代码逻辑运行在子进程中也就是SystemServer进程中执行
    if (pid == 0) {
    if (hasSecondZygote(abiList)) {
        waitForSecondaryZygote(socketName);
    }
    //3. 为什么把socket关闭了呢？因为SystemServer是通过zygote fork进程,复制了zygote进程的地址空间，
    //也拥有zygote进程创建的socket：zygoteServer
    //但是SystemServer不需要socket通信的 所以将socket关闭了
    zygoteServer.closeServerSocket();
    //4. 处理SystemServer进程 返回了一个Runnable类型
    return handleSystemServerProcess(parsedArgs);
}
```

- 注释1: 调用了`native`方法通过JNI调用了`nativeForkSystemServer`� 创建`SystemServer`进程
- 注释2: pid = 0说明当前的**代码是运行在新创建的子进程中**的，也就是运行在SystemServer进程中。
- 注释3: 因为SystemServer是被Zygote进程fork，所以SystemServer进程复制了Zygote进程的地址空间，拥有ZygoteServer，但是SystemServer不需要socket通信，所以将socket关闭了,避免资源浪费。
- 注释4: 处理SystemServer进程

下面看`handleSystemServerProcess`做了些什么操作？
```cpp
ClassLoader cl = null;
if (systemServerClasspath != null) {
    //1. pathClassLoader的创建，用于调用SystemServer的class
    cl = createPathClassLoader(systemServerClasspath, parsedArgs.mTargetSdkVersion);

    Thread.currentThread().setContextClassLoader(cl);
}

//2. 调用zygoteInit方法
return ZygoteInit.zygoteInit(parsedArgs.mTargetSdkVersion,
    parsedArgs.mDisabledCompatChanges,
    parsedArgs.mRemainingArgs, cl);
```

- 注释1: 创建了一个`PathClassLoader` 去加载`SystemServer.class`, 至于`PathClassLoader` 会在后续单独的文章介绍Android/Java的`ClassLoader`
- 注释2: 调用了`zygoteInit()`方法，这个方法启动`SystemServer`进程的核心方法。需要注意两点：将`classLoader`参数传递进去了; 方法返回了`Runnable`类型；

`zygoteInit`方法实现如下：
```cpp
//1. 启动binder线程池
ZygoteInit.nativeZygoteInit();
//2. 进入SystemServer的main方法
return RuntimeInit.applicationInit(targetSdkVersion, disabledCompatChanges, argv,
    classLoader);
```

- 注释1: 启动了`binder`线程池调用`nativeZygoteInit`方法,如下代码`nativeZygoteInit`没有实现，其实调用了`JNI`的方法，然后调用了`Native`层启动了`binder`线程池，`SystemServer`进程就可以使用`binder`实现进程间的通信了。
```cpp
private static final native void nativeZygoteInit();

```
那么问题来了 `nativeZygoteInit` 的JNI的实现在哪里呢？还记得在上一篇zygote启动原理中，zygote进程启动过程，简单的回忆一下：

1. 调用`AndroidRuntime`的`start`函数，启动`Zygote`进程
2. 创建Java虚拟机并为Java虚拟机注册JNI方法
3. 通过JNI调用`ZygoteInit`的`main`方法进入Zygote的Java框架层
4. 创建服务端`Socket`，并通过`runSelectLoop`方法等待AMS的请求来创建新的应用程序进程
5. 启动`SystemServer`进程

来看`Zygote`进程中在`AndroidRuntime.start`方法中注册了JNI方法，看看有哪些JNI方法：
```cpp
/*static*/ int AndroidRuntime::startReg(JNIEnv* env)
{
    //gRegJNI 获取要注册的JNI方法
    if (register_jni_procs(gRegJNI, NELEM(gRegJNI), env) < 0) {
        env->PopLocalFrame(NULL);
        return -1;
    }
    env->PopLocalFrame(NULL);
    return 0;
}
```
`gRegJNI`方法获取将要注册的JNI方法,在第二行就可以看到`register_com_android_internal_os_ZygoteInit_nativeZygoteInit`，其实就是`nativeZygoteInit`的方法的注册
```cpp
static const RegJNIRec gRegJNI[] = {
        REG_JNI(register_com_android_internal_os_RuntimeInit),
        REG_JNI(register_com_android_internal_os_ZygoteInit_nativeZygoteInit)
        ......
}
```
实现如下：主要技术点就是JNI记录Java的Native方法和JNI的关联关系
```cpp
int register_com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env)
{
    //JNI 记录Java的native方法和JNI的关联的关系
    const JNINativeMethod methods[] = {
        { "nativeZygoteInit", "()V",
            (void*) com_android_internal_os_ZygoteInit_nativeZygoteInit },
    };
    return jniRegisterNativeMethods(env, "com/android/internal/os/ZygoteInit",
        methods, NELEM(methods));
}

```
也就是说Java的`nativeZygoteInit`方法对应的JNI的方法是`com_android_internal_os_ZygoteInit_nativeZygoteInit`实现如下：
```cpp
static AndroidRuntime* gCurRuntime = NULL;
static void com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env, jobject clazz)
{
    gCurRuntime->onZygoteInit();
}
```
在上述代码中调用了`AndroidRuntime.onZygoteInit()`方法，但是`AndroidRuntime`没有实现该方法，是由子类`AppRuntime`实现，`AppRuntime`在`app_main.cpp`中，如下代码最终启动了binder的线程池，关于binder相关的大技术点，后续需要几篇的文章讲解。
```cpp
class AppRuntime : public AndroidRuntime
{
   virtual void onZygoteInit()
    {
        sp<ProcessState> proc = ProcessState::self();
        ALOGV("App process: starting thread pool.\n");
        //启动binder线程池,通过binder与其他进程通信
        proc->startThreadPool();
    }
}
```
在回到zygoteInit()方法:
```cpp
//1. 启动binder线程池
ZygoteInit.nativeZygoteInit();
//2. 进入SystemServer的main方法
return RuntimeInit.applicationInit(targetSdkVersion, disabledCompatChanges, argv,
    classLoader);
```

- 注释2: 主要的作用是通过`PathClassLoader`调用了`SystemServer.main()`方法，`applicationInit`的实现如下：

最终调用了`findStaticMain`方法，顾名思义就是找到`SystemServer`类的`main`静态方法
```cpp
 protected static Runnable findStaticMain(String className, String[] argv,
            ClassLoader classLoader) {
        Class<?> cl;
        try {
            //通过反射得到SystemServer类
            cl = Class.forName(className, true, classLoader);
        } catch (ClassNotFoundException ex) {
            throw new RuntimeException(
                    "Missing class when invoking static main " + className,
                    ex);
        }

        Method m;
        try {
            //找到main方法
            m = cl.getMethod("main", new Class[] { String[].class });
        } catch (NoSuchMethodException ex) {
            throw new RuntimeException(
                    "Missing static main on " + className, ex);
        } catch (SecurityException ex) {
            throw new RuntimeException(
                    "Problem getting static main on " + className, ex);
        }

        int modifiers = m.getModifiers();
        return new MethodAndArgsCaller(m, argv);
    }
```
上述代码 返回了MethodAndArgsCaller的实例，大胆猜测一下，实现了Runnable接口，通过run方法调用了main方法
```cpp
    static class MethodAndArgsCaller implements Runnable {
        private final Method mMethod;

        private final String[] mArgs;

        public MethodAndArgsCaller(Method method, String[] args) {
            mMethod = method;
            mArgs = args;
        }

        public void run() {
            //调用了SystemServer的main方法中
            mMethod.invoke(null, new Object[] { mArgs });
        }
    }
```
回到疑问：返回Runnable类型，这是为什么呢？其实就是等待SystemServer进程处理完毕，然后调用run方法启动SystemServer进程。
```cpp
Runnable r = forkSystemServer(abiList, zygoteSocketName, zygoteServer);
// 调用SystemServer.main()方法
if (r != null) {
    r.run();
    return;
}
```
在Zygote启动SystemServer进程过程中主要做了

- fork SystemServer进程
- 启动Binder线程池，这样SystemServer进程就可以使用binder与其他进程进行通讯
- 调用SystemServer.main方法，启动SystemServer进程
### 解析SystemServer进程
> 在上述讲解中，通过调用SystemServer.main方法启动了SystemServer进程的工作内容

代码如下：

- 注释1: 创建了`SystemServiceManger` 对系统服务的管理类，对系统服务的创建、启动等管理
- 注释2: 分别启动系统服务：引导服务、核心服务、其他服务
```cpp
public static void main(String[] args) {
    new SystemServer().run();
}

private void run() {
    //1. SystemServiceManager 对系统服务的管理类
    mSystemServiceManager = new SystemServiceManager(mSystemContext);
    mSystemServiceManager.setStartInfo(mRuntimeRestart,
        mRuntimeStartElapsedTime, mRuntimeStartUptime);
    //LocalServices 保存本地的service 保存的service都在同一进程
    LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
    2. 启动服务
    startBootstrapServices(t);//启动引导服务
    startCoreServices(t);//启动核心服务
    startOtherServices(t);//启动其他服务
}
```

- 引导服务有如下服务

![image.png](https://cdn.nlark.com/yuque/0/2023/png/375694/1685440486267-6fc20855-b0ca-488c-bc7a-a963c5cb9238.png#averageHue=%23dadada&clientId=u25325749-d307-4&from=paste&height=521&id=u56dc60ea&originHeight=1042&originWidth=2442&originalType=binary&ratio=2&rotation=0&showTitle=false&size=460461&status=done&style=none&taskId=u31d2ea2d-352b-419c-a32e-57491145833&title=&width=1221)

- 核心服务如下：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/375694/1685440536648-eadcbdd3-f107-4c53-8c98-16e62d9b87ae.png#averageHue=%23d8d8d8&clientId=u25325749-d307-4&from=paste&height=354&id=ud9640acf&originHeight=708&originWidth=2410&originalType=binary&ratio=2&rotation=0&showTitle=false&size=213578&status=done&style=none&taskId=u0b811401-f4ff-45a7-824d-29e015cf70e&title=&width=1205)

- 其他服务如下：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/375694/1685440572165-c95f584a-d823-401f-954a-996ba895dfc1.png#averageHue=%23e2e2e2&clientId=u25325749-d307-4&from=paste&height=631&id=udc853671&originHeight=1262&originWidth=2290&originalType=binary&ratio=2&rotation=0&showTitle=false&size=423187&status=done&style=none&taskId=u63a6f8d7-e8b2-4b5e-9713-3d3f1cb00fc&title=&width=1145)
通过上述代码，可以知道启动服务就是通过`SystemServiceManger`进程启动的如下代码：
> 最终启动服务都会调用`SystemService`具体实现类的`onStart`方法去启动服务

```cpp
mSystemServiceManager.startService(new SensorPrivacyService(mSystemContext));


public void startService(@NonNull final SystemService service) {
    // Register it.
    // 注册service
    mServices.add(service);

    //启动service
    service.onStart();
}
```
### 总结
通过上述的学习，可以清楚的知道SystemServer进程主要做了如下工作：

1. 启动Binder线程池，与其他进程进行通信
2. 创建SystemServiceManager用于对系统的服务进行创建、启动和生命周期管理
3. 启动各种系统服务：引导服务、核心服务、其他服务

SystemServer的启动原理时序图如下：
![](./SystemServer%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8%E5%88%86%E6%9E%90.assets/1685608509695-e7141a2e-c3e7-452a-8fb7-9a397dfe04fb.jpeg)