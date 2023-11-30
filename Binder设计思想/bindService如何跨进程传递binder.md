> bindService的过程其实就是Activity和Service跨进程通信的过程，这是理解Binder的里程碑式的阶段，如果你明白了《Binder通信机制》和《AIDL的跨进程通信原理》以及《bindService》，对于Java层的Binder将彻底掌握本质。

通过前几篇文章，可以清楚的知道Client端可以通过`ServiceManager`获取到Server的binder，然后进行通信，那么在上一篇文章中的AIDL跨进程通信原理，**Client端是通过ServiceManager拿到的binder吗？**显然不是因为Server端的binder并没有注册到`ServiceManager`中，不能直接通信这里涉及到权限管理问题需要通过`AMS`来完成。

下面来看bindService做了什么如何将binder传递给Client端(这里需要你来打开Android的源码)：

按照惯例线上流程图：

1. Client端和ServiceManager通信获得AMS的IBinder

![image-20231024131847984](./bindService%E5%A6%82%E4%BD%95%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92binder.assets/image-20231024131847984.png)

2. Client和AMS通信执行bindService请求，AMS与Server端通信执行Service的onBind获取Server的IBinder

![image-20231024132044886](./bindService%E5%A6%82%E4%BD%95%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92binder.assets/image-20231024132044886.png)

在Activity代码中调用了bindService:

```kotlin
val intent = Intent(this, AidlService::class.java)
bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE)
```

bindService的真正实现代码实在`ContextImpl`中，调用了`bindServiceCommon`：

```java
@Override
public boolean bindService(Intent service, ServiceConnection conn, int flags) {
warnIfCallingFromSystemProcess();
return bindServiceCommon(service, conn, flags, null, mMainThread.getHandler(), null,
getUser());
}
```

而在`bindServiceCommon`中的核心代码就是调用了`ActivityManager.getService()`,实际上会调用到AMS：

```java
int res = ActivityManager.getService().bindIsolatedService(
mMainThread.getApplicationThread(), getActivityToken(), service,
service.resolveTypeIfNeeded(getContentResolver()),
sd, flags, instanceName, getOpPackageName(), user.getIdentifier());
```

下面来看ActivityManager.getService()是如何获取服务的如下代码：

> 这里可以看到`ServiceManager.getService`就是从`ServiceManager`中获取注册的AMS的binder。

这里还会看到一段熟悉的代码：`IActivityManager.Stub.asInterface(b)` 这不就是前面写的AIDL的例子吗？Client获取Server的`Proxy`对象。

```java
@UnsupportedAppUsage
public static IActivityManager getService() {
	return IActivityManagerSingleton.get();
}
@UnsupportedAppUsage
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

那么就可以猜测AMS服务就是使用AIDL来进行跨进程通信的，可以在源码中找到`IActivityManager.aidl`文件

![image-20231024105845210](./bindService%E5%A6%82%E4%BD%95%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92binder.assets/image-20231024105845210.png)

那么AMS作为server端一定实现了Stub：

```java
public class ActivityManagerService extends IActivityManager.Stub
```

看到这里应该就能彻底理解AMS就是通过AIDL进行跨进程通信的，在回到源码中会调用AMS的`bindIsolatedService`方法：

```java
public int bindIsolatedService(IApplicationThread caller, IBinder token, Intent service,
                               String resolvedType, IServiceConnection connection, int flags, String instanceName,
                               String callingPackage, int userId) throws TransactionTooLargeException {
    ....
        synchronized(this) {
        return mServices.bindServiceLocked(caller, token, service,
                                           resolvedType, connection, flags, instanceName, callingPackage, userId);
    }
}
```

`bindServiceLocked`方法会调用`requestServiceBindingLocked`方法核心代码如下：

```java
r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind,
                                 r.app.getReportedProcState());
```

这里的thread是`IApplicationThread`，从源码来看是报红的，那么就可以猜测`IApplicationThread`其实是`IApplicationThread.aidl`生成的Java文件，那么Stub实现类就是`ApplicationThread`

![image-20231024110624761](./bindService%E5%A6%82%E4%BD%95%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92binder.assets/image-20231024110624761.png)

如下代码果然如此，`scheduleBindService` 方法其实就是**AMS进程**调用**App进程**的一次跨进程通信

```java
private class ApplicationThread extends IApplicationThread.Stub{
    public final void scheduleBindService(IBinder token, Intent intent,
                                          boolean rebind, int processState) {
        updateProcessState(processState, false);
        BindServiceData s = new BindServiceData();
        s.token = token;
        s.intent = intent;
        s.rebind = rebind;
        sendMessage(H.BIND_SERVICE, s);
    }
}
```

可以看到发送了Handler的消息，调用了`handleBindService`方法核心代码如下：

调用了`Service.onBind()`方法，这里可以知道在前面的例子中这个方法返回了`IBinder`,这个就是Server端的binder，这里又进行了一次跨进程通信调用AMS的`publishService`将binder传递给了AMS

```java
Service s = mServices.get(data.token);
IBinder binder = s.onBind(data.intent);//获取server的binder
ActivityManager.getService().publishService(
data.token, data.intent, binder);
```

再来看AMS的publishService方法调用了`publishServiceLocked`方法核心代码：

```java
 c.conn.connected(r.name, service, false);
```

`conn`是`IServiceConnection`又是一个aidl跨进程通信，将service就是Server端的binder。

![image-20231024111838952](./bindService%E5%A6%82%E4%BD%95%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92binder.assets/image-20231024111838952.png)

那么如何调用`ServiceConnection`的回调方法`onServiceConnected`的呢？其实在一开始的`bindServiceCommon`方法中：

将`ServiceConnection`实例传递给了`getServiceDispatcher`方法并返回了`IServiceConnection`

```java
private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags,
                                  String instanceName, Handler handler, Executor executor, UserHandle user) {
    IServiceConnection sd;
    if (executor != null) {
        sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(), executor, flags);
    } else {
        sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(), handler, flags);
    }
}
```

最终在`LoadedApk`中的内部类`ServiceDispatcher`的内部类`InnerConnection`找到了IServiceConnection.Stub的实现

![image-20231024113102410](./bindService%E5%A6%82%E4%BD%95%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92binder.assets/image-20231024113102410.png)

```java
private static class InnerConnection extends IServiceConnection.Stub {
    @UnsupportedAppUsage
    final WeakReference<LoadedApk.ServiceDispatcher> mDispatcher;

    InnerConnection(LoadedApk.ServiceDispatcher sd) {
        mDispatcher = new WeakReference<LoadedApk.ServiceDispatcher>(sd);
    }

    public void connected(ComponentName name, IBinder service, boolean dead)
        throws RemoteException {
        LoadedApk.ServiceDispatcher sd = mDispatcher.get();
        if (sd != null) {
            sd.connected(name, service, dead);
        }
    }
}
```

当调用`connected`方法时候，最终会调用到`doConnected`方法,RunConnection中也会最终调用`doConnected`方法：

```java
public void connected(ComponentName name, IBinder service, boolean dead) {
    if (mActivityExecutor != null) {
        mActivityExecutor.execute(new RunConnection(name, service, 0, dead));
    } else if (mActivityThread != null) {
        mActivityThread.post(new RunConnection(name, service, 0, dead));
    } else {
        doConnected(name, service, dead);
    }
}
```

`RunConnection`的`run`方法：

```java
public void run() {
    if (mCommand == 0) {
        doConnected(mName, mService, mDead);
    } else if (mCommand == 1) {
        doDeath(mName, mService);
    }
}
```

在这里就看到了会调用 `onServiceConnected` 至此整个`bindService`流程结束。

```java
public void doConnected(ComponentName name, IBinder service, boolean dead) {
    if (old != null) {
        mConnection.onServiceDisconnected(name);
    }
    if (dead) {
        mConnection.onBindingDied(name);
    }
    // If there is a new viable service, it is now connected.
    if (service != null) {
        mConnection.onServiceConnected(name, service);
    } else {
        // The binding machinery worked, but the remote returned null from onBind().
        mConnection.onNullBinding(name);
    }
}
```

整个bindService的流程执行了`8次跨进程通信`，Client端才拿到了Server端的IBinder:

1. Client 端和`ServiceManager` 通信是一次跨进程通信
2. 获取AMS的`IBinder` 是一次跨进程通信
3. 执行`AMS.bindService()` 跨进程通信
4. AMS调用`scheduleBindService()`调用Server端跨进程通信
5. Server端和`ServiceManager`通信
6. `ServiceManager`返回AMS的IBinder跨进程通信
7. Server端调用AMS的`publishService`跨进程通信
8. AMS调用Client端`InnerConnection.connected`跨进程通信最终将Server端的`IBinder`传递到`onServiceConnected()`

![image-20231024132228895](./bindService%E5%A6%82%E4%BD%95%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92binder.assets/image-20231024132228895.png)

