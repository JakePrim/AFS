# AIDL 跨进程通信的原理

> 上一篇文章《进程间通信的几种方式》除了socket基本上都是通过Binder进行通信的，在开发中常用的跨进程通信方式就是AIDL，搞懂AIDL跨进程通信的原理是非常有必要的。包括AMS等服务跨进程通信都是使用了AIDL运用Binder实现跨进程通信 。(PS:Android 29 之后提供了aidl.exe 命令 生成对应的AIDL的cpp文件) 阅读该文章前提：《Binder通信机制》

上一篇文章中使用AIDL 会生成java文件：

![](./AIDL%20%E8%B7%A8%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A1%E5%8E%9F%E7%90%86.assets/Snipaste_2023-10-23_15-58-34.png)

`Stub`: server端交互,继承了Binder，实现IInterface接口，就是服务端的Binder

`Binder` 驱动：和Stub打交道，跨进程通信，请求服务端Stub的实现

`Proxy`: 服务端能够提供服务的代理类，给客户端用。代理的就是Stub

AIDL的核心原理如下图所示：

1. `aidl`文件会生成`IInterface.java`文件,生成的java文件内部类`Stub(Binder)`
2. Server端实现`Stub`中的IInterface接口方法，`Stub`实例就是Binder实例
3. Server端通过`onBinder`方法返回Binder实例
4. Client通过`bindService`的回调获得`IBinder`
5. 通过`Stub.asInterface(binder)`方法获取IInterface实例，这个实例其实是Stub的内部类`Proxy`,`Proxy`类实现了IInterface接口，`Proxy`类通过binder进行通信。

![image-20231023161055965](./AIDL%20%E8%B7%A8%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A1%E5%8E%9F%E7%90%86.assets/image-20231023161055965.png)

其实AIDL的核心代码就在于Stub和Proxy的实现：

Stub 其实就是Binder,AIDL生成的Java文件代码如下：

Stub继承了Binder并`implements IGameInterface`接口，返回了Stub.Proxy的内部类

```java
public static abstract class Stub extends android.os.Binder implements com.example.myapplication.IGameInterface{
    public static com.example.myapplication.IGameInterface asInterface(android.os.IBinder obj)
    {
        if ((obj==null)) {
            return null;
        }
        android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
        if (((iin!=null)&&(iin instanceof com.example.myapplication.IGameInterface))) {
            return ((com.example.myapplication.IGameInterface)iin);
        }
        return new com.example.myapplication.IGameInterface.Stub.Proxy(obj);
    }
}
```

Stub实例是在server端创建实例的，通过onBind将IBinder返回：

```kotlin
//创建Binder
private val binder = object : IGameInterface.Stub() {
    override fun addGame(game: Game) {
        games.add(game)
    }

    override fun getGameList(): MutableList<Game> {
        games.add(Game("巫师3", "巫师三 ----"))
        games.add(Game("星空", "B社 星空"))
        return games
    }
}

override fun onBind(intent: Intent?): IBinder {
    return binder
}
```

Client和Server通信的关键在于`Proxy`类,如下代码：

通过构造函数将`IBinder`传入，然后通过binder和Server端跨进程通信:

```java
private static class Proxy implements com.example.myapplication.IGameInterface
{
    private android.os.IBinder mRemote;
    Proxy(android.os.IBinder remote)
    {
        mRemote = remote;
    }
    @Override public void addGame(com.example.myapplication.Game game) throws android.os.RemoteException
    {
        ///.....
        boolean _status = mRemote.transact(Stub.TRANSACTION_addGame, _data, _reply, 0);
        ///......
    }
}
```

Client端通过`bindService`的回调，获取Binder如下代码：

`Stub.asInterface(binder)` 就是调用了`Proxy(binder)`

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val intent = Intent(this, AidlService::class.java)
    bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE)
}

private val mServiceConnection = object : ServiceConnection {
    override fun onServiceConnected(p0: ComponentName?, binder: IBinder?) {
        //通过Binder获取IGameInterface
        gameBinder = IGameInterface.Stub.asInterface(binder)
        gameBinder?.addGame(Game("刺客信条", "信条"))
        gameBinder?.gameList?.forEach {
            Log.e("TAG", "onServiceConnected-> name:${it.name}  info:${it.info}")
        }
    }

    override fun onServiceDisconnected(p0: ComponentName?) {
        gameBinder = null
    }
}
```

> 如上述所讲就是整个AIDL的跨进程通信过程，整个流程很简单，其实AIDL生成的Java文件就是帮助开发者更方便的实现跨进程通信。

这里还有一个疑问，`Client`端通过`bindService`是如何拿到Binder的？如何将Binder传递给Client端的呢？bindService也是跨进程通信的吗？下一篇文章详细讲解











