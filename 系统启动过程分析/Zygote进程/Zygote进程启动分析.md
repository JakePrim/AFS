# Zygote 进程启动过程分析

在上一篇文章中讲解了init进程的启动原理和主要做了哪些工作。
[Framework | Init进程启动原理](https://www.yuque.com/jakeprim/android/gmgsed?view=doc_embed)

> 在`Android`系统中，`DVM`和`ART`、`应用程序`进程以及`SystemServer`进程都是由`Zygote`进程来创建的，所以`Zygote`进程也称为孵化器，它通过`fork`的形式来创建`应用程序`进程和`SystemServer`进程，由于`Zygote`进程在启动时会创建`DVM`和`ART`，因此通过`fork`而创建的`应用程序`进程和`SystemServer`进程可以在内部获取一个`DVM`或者`ART`的实例拷贝。
> Zygote进程是在init进程启动时创建的，最开始Zygote进程的名称并不是叫"zygote"而是叫"app_process"，这个名称是在Android.mk中定义的，Zygote进程启动后，Linux系统下的pctrl系统会调用app_proccess，将其名称换成了“zygote".

Zygote 启动脚本,启动不同版本的rc文件,从Android5.0开始Android开始支持64位
```cpp
...
import /system/etc/init/hw/init.${ro.zygote}.rc
```
脚本文件放在了`system/core/rootdir`目录中

- `init.zygote32.rc`
- `init.zygote32_64.rc`
- `init.zygote64.rc`
- `init.zygote64_32.rc`
```cpp
service zygote /system/bin/app_process32 -Xzygote /system/bin --zygote --start-system-server --socket-name=zygote
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote stream 660 root system
    socket usap_pool_primary stream 660 root system
    //onrestart 如果进程被杀死 重启进程
    onrestart exec_background - system system -- /system/bin/vdc volume abort_fuse
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks

service zygote_secondary /system/bin/app_process64 -Xzygote /system/bin --zygote --socket-name=zygote_secondary
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote_secondary stream 660 root system
    socket usap_pool_secondary stream 660 root system
    onrestart restart zygote
    writepid /dev/cpuset/foreground/tasks

```

### Zygote进程启动
在上一篇的init进程启动原理中，知道了init进程fork启动了zygote进程，调用了`app_main.cpp`中的`main`函数,如下代码所示：
> 调用runtime.start启动zygote进程，runtime是AndroidRuntime.cpp

```cpp
int main(int argc, char* const argv[])
{
    ...
    //zygote都是fork自身来创建子进程 那么每次fork子进程都会进入到main函数 所以需要进行判断
    //当前运行的进程
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {//1
            //如果当前运行在zygote进程，则将zygote设置为true
            zygote = true;//2
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {//3
            //如果当前运行在SystemServer进程，则将startSystemServer设置为true
            startSystemServer = true;//4
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }
   ...
    if (zygote) {//5 如果zygote为true表示运行在zygote进程中
        //AndroidRuntime
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
    }     
}
```
如下代码声明了`AppRuntime`,实际上调用了`AndroidRuntime`的`start`函数
```cpp
class AppRuntime : public AndroidRuntime
```
start的代码逻辑如下主要做了JNI的操作：

1. 创建JNIEnv用来和Java层进行交互
2. 启动JVM虚拟机和为虚拟机注册JNI方法
3. 找到ZygoteInit类调用main函数，启动zygote进程，并进入到了Java框架层
```cpp
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
    ...
    //1. 创建了JNIEnv 可以和java代码进行交互
    JNIEnv* env;
    //2. 启动Java虚拟机
    if (startVm(&mJavaVM, &env, zygote, primary_zygote) != 0) {
        return;
    }
    onVmCreated(env);
    if (startReg(env) < 0) {//3. 为Java虚拟机 注册了JNI方法
        ALOGE("Unable to register all android natives\n");
        return;
    }
        ...
/** 4. 从app_main的main函数得知className为com.android.internal.os.ZygoteInit
将其转换成JNI的String类型
**/
    classNameStr = env->NewStringUTF(className);
    ...
    //4. 内部会将className中的.替换成/
    char* slashClassName = toSlashClassName(className != NULL ? className : "");
    //5. 得到ZygoteInit类
    jclass startClass = env->FindClass(slashClassName);
    if (startClass == NULL) {
        ALOGE("JavaVM unable to locate class '%s'\n", slashClassName);
    } else {
        //6. 找到ZygoteInit的main方法
        jmethodID startMeth = env->GetStaticMethodID(startClass, "main",
        "([Ljava/lang/String;)V");
        if (startMeth == NULL) {
            ALOGE("JavaVM unable to find main() in '%s'\n", className);
            /* keep going */
        } else {
            //7. 通过JNI调用ZygoteInit的main方法，这里进入到Java框架层 启动了Zygote进程
            env->CallStaticVoidMethod(startClass, startMeth, strArray);

            [[if]] 0
            if (env->ExceptionCheck())
                threadExitUncaughtException(env);
            [[endif]]
        }
    }
}
```
下面看ZygoteInit.java主要做了哪些工作：

1. 创建ZygoteServer socket服务端，用于接收AMS请求创建应用进程
2. fork systemServer进程
3. 等待AMS请求
```cpp
    public static void main(String argv[]) {
        //1. socket server服务端
        ZygoteServer zygoteServer = null;
        ...
         //1. 创建了ZygoteServer内部创建一个server端的socket，用于等待AMS请求socket来创建应用程序进程
            zygoteServer = new ZygoteServer(isPrimaryZygote);

            if (startSystemServer) {
                //2. fork systemServer进程
                Runnable r = forkSystemServer(abiList, zygoteSocketName, zygoteServer);

                // {@code r == null} in the parent (zygote) process, and {@code r != null} in the
                // child (system_server) process.
                if (r != null) {
                    r.run();
                    return;
                }
            }

            Log.i(TAG, "Accepting command socket connections");

            // The select loop returns early in the child process after a fork and
            // loops forever in the zygote.
            //3. 开始轮询，等待AMS请求
            caller = zygoteServer.runSelectLoop(abiList);
        ...
    }
```
在上述代码中，创建了一个服务端的socket用于接收AMS的请求，forkSystemServer 创建了SystemServer进程, 下面看看如何创建SystemServer进程的。(注意现在的代码是在Java框架层的)
```cpp
private static Runnable forkSystemServer(String abiList, String socketName,
            ZygoteServer zygoteServer) {
        //1. 创建args数组，这个数组用来保存启动SystemServer启动参数
        String args[] = {
                "--setuid=1000",
                "--setgid=1000",
                "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1023,"
                        + "1024,1032,1065,3001,3002,3003,3006,3007,3009,3010,3011",
                "--capabilities=" + capabilities + "," + capabilities,
                "--nice-name=system_server",
                "--runtime-args",
                "--target-sdk-version=" + VMRuntime.SDK_VERSION_CUR_DEVELOPMENT,
                "com.android.server.SystemServer",
        };
        ZygoteArguments parsedArgs = null;
        ...
        parsedArgs = new ZygoteArguments(args);//2
        //2. 创建一个子进程SystemServer
        pid = Zygote.forkSystemServer(
                    parsedArgs.mUid, parsedArgs.mGid,
                    parsedArgs.mGids,
                    parsedArgs.mRuntimeFlags,
                    null,
                    parsedArgs.mPermittedCapabilities,
                    parsedArgs.mEffectiveCapabilities);
        ...
        //当前代码逻辑运行在子进程中
        if (pid == 0) {
            if (hasSecondZygote(abiList)) {
                waitForSecondaryZygote(socketName);
            }

            zygoteServer.closeServerSocket();
            //4. 处理SystemServer进程
            return handleSystemServerProcess(parsedArgs);
        }
    }
```
调用了Zygote.forkSystemServer创建的SystemServer进程，最终是调用的native层fork创建的SystemServer进程
```cpp
private static native int nativeForkSystemServer(int uid, int gid, int[] gids, int runtimeFlags,
            int[][] rlimits, long permittedCapabilities, long effectiveCapabilities);
```
关于SystemServer 进程的启动过程会在下一篇文章中详解。
### 如何接收AMS请求并创建应用程序进程
> 在上述代码中通过Zygote.runSelectLoop(),轮询等待AMS请求。开启一个无限循环
> 思考一个问题 为什么返回类型是Runnable类型？这个问题会在后面的文章中给出答案

```cpp
    Runnable runSelectLoop(String abiList) {
        ...
         ArrayList<FileDescriptor> socketFDs = new ArrayList<>();
        ArrayList<ZygoteConnection> peers = new ArrayList<>();

        //1. mZygoteSocket socket 服务端，getFileDescriptor 获取该socket的文件描述符
        socketFDs.add(mZygoteSocket.getFileDescriptor());//1.-->
        while (true) {//开启无限循环      
            ...
            StructPollfd[] pollFDs;
            ...
             int pollIndex = 0;
            //遍历socketFDs的信息 转移StructPollfd
            //events 请求的事件
            //revents 返回的事件
            //fd 文件描述符
            for (FileDescriptor socketFD : socketFDs) {//2.
                pollFDs[pollIndex] = new StructPollfd();
                pollFDs[pollIndex].fd = socketFD;
                pollFDs[pollIndex].events = (short) POLLIN;
                ++pollIndex;
            }
        ...
        int pollReturnValue;
            try {
            //poll 函数可以监听多个文件描述符 等待文件描述符上的POLLIN事件，代码会阻塞在这里直到响应之后才继续往下执行
                pollReturnValue = Os.poll(pollFDs, pollTimeoutMs);
            } catch (ErrnoException ex) {
                throw new RuntimeException("poll failed", ex);
            }

            if (pollReturnValue == 0) {
                //
                mUsapPoolRefillTriggerTimestamp = INVALID_TIMESTAMP;
                mUsapPoolRefillAction = UsapPoolRefillAction.DELAYED;
            } else {    
                 while (--pollIndex >= 0) {
                    //如果没有实际发生的事件并不是可读事件就跳过该文件描述符
                    if ((pollFDs[pollIndex].revents & POLLIN) == 0) {
                        continue;
                    }
                    if (pollIndex == 0) {//创建一个连接
                        // Zygote server socket
                        ZygoteConnection newPeer = acceptCommandPeer(abiList);
                        peers.add(newPeer);
                        socketFDs.add(newPeer.getFileDescriptor());

                    } else if (pollIndex < usapPoolEventFDIndex) {
                         //4. 通过ZygoteConnection创建一个新的应用程序进程
                        ZygoteConnection connection = peers.get(pollIndex);
                        final Runnable command = connection.processOneCommand(this);
                        if (mIsForkChild) {
                            ...
                            return command;
                        }
                          ...  
            }     
        }     
        ...
    }
```
上述代码和Loop.loop()很像，在OS.poll()函数进行了阻塞。上述代码比较多 一个个拆分解读。
#### StructPollfd和FileDescriptor
在runSelectLoop()函数中出现两个类型：FileDescriptor文件描述符也可以叫FD，简单描述的话他是一个索引值(非负整数)，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或创建一个新文件时，内核向进程返回一个文件描述符。在Linux内核对所有打开的文件有一个文件描述符表格，里面存储了每个文件描述符作为索引与一个打开文件相对应的关系，文件描述符(索引)就是文件描述符表的下标，数组的内容就是一个个打开文件的指针。
StructPollfd:等待文件描述符的事件对象
```cpp
public final class StructPollfd {
    public short events;//events 代表了关注的事件
    public FileDescriptor fd;// fd代表了文件描述符
    public short revents;//revents 代表了实际发生的事件
    public Object userData;
}
```
那么在上述代码：StructPollfd[] 数组 对events赋值POLLIN 事件，那么poll函数就是等待POLLIN事件。
```cpp
for (FileDescriptor socketFD : socketFDs) {//2.
                pollFDs[pollIndex] = new StructPollfd();
                pollFDs[pollIndex].fd = socketFD;
                pollFDs[pollIndex].events = (short) POLLIN;
                ++pollIndex;
            }
```

下面有几个详细的参考链接：
[https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6)
[彻底弄懂 Linux 下的文件描述符（fd） | 半亩方塘](https://yushuaige.github.io/2020/08/14/%E5%BD%BB%E5%BA%95%E5%BC%84%E6%87%82%20Linux%20%E4%B8%8B%E7%9A%84%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6%EF%BC%88fd%EF%BC%89/)

#### OS.poll()
> `OS.poll()`函数，使`runSelectLoop()`函数阻塞起来，`poll`方法调用了`Linux`底层，等待文件描述符的`POLLIN`事件

```cpp
Os.poll(pollFDs, pollTimeoutMs);
```

#### 创建连接
> 当`pollIndex == 0`的时候调用`acceptCommandPeer`创建了一个连接。

```cpp
if (pollIndex == 0) {//没有可处理的fd 创建一个连接
                        // Zygote server socket

                        ZygoteConnection newPeer = acceptCommandPeer(abiList);
                        peers.add(newPeer);
                        socketFDs.add(newPeer.getFileDescriptor());

 } 
```
创建socket连接的核心代码如下：最终调用了Os.accept()根据参数FD创建一个新的Socket连接并返回该连接的FD。
```cpp
class ZygoteServer {
    private ZygoteConnection acceptCommandPeer(String abiList) {
        try {
            // mZygoteSocket的类型为LocalServerSocket
            return createNewConnection(mZygoteSocket.accept(), abiList);
        } catch (IOException ex) {
            throw new RuntimeException(
                    "IOException during accept()", ex);
        }
    }
    
    protected ZygoteConnection createNewConnection(LocalSocket socket, String abiList)
            throws IOException {
        return new ZygoteConnection(socket, abiList);
    }
}


public class LocalServerSocket implements Closeable {

    private final LocalSocketImpl impl;
    
    public LocalSocket accept() throws IOException
    {
        LocalSocketImpl acceptedImpl = new LocalSocketImpl();

        impl.accept(acceptedImpl);

        return LocalSocket.createLocalSocketForAccept(acceptedImpl);
    }
}

class LocalSocketImpl {
    
    protected void accept(LocalSocketImpl s) throws IOException {
        if (fd == null) {
            throw new IOException("socket not created");
        }

        try {
            //重点就是这行Os.accept
            s.fd = Os.accept(fd, null /* address */);
            s.mFdCreatedInternally = true;
        } catch (ErrnoException e) {
            throw e.rethrowAsIOException();
        }
    }
}

```

当第二次监听到POLLIN事件就会触发`pollIndex < usapPoolEventFDIndex`通过`ZygoteConnection.processOneCommand`创建的应用程序进程。
```cpp
 Runnable processOneCommand(ZygoteServer zygoteServer) {
     ...
     //创建子进程 调用了native的nativeForkAndSpecialize方法
     pid = Zygote.forkAndSpecialize(parsedArgs.mUid, parsedArgs.mGid, parsedArgs.mGids,
                parsedArgs.mRuntimeFlags, rlimits, parsedArgs.mMountExternal, parsedArgs.mSeInfo,
                parsedArgs.mNiceName, fdsToClose, fdsToIgnore, parsedArgs.mStartChildZygote,
                parsedArgs.mInstructionSet, parsedArgs.mAppDataDir, parsedArgs.mIsTopApp,
                parsedArgs.mPkgDataInfoList, parsedArgs.mWhitelistedDataInfoList,
                parsedArgs.mBindMountAppDataDirs, parsedArgs.mBindMountAppStorageDirs);

        try {
            if (pid == 0) {
                // in child 告诉socket子进程已经创建
                zygoteServer.setForkChild();

                zygoteServer.closeServerSocket();
                IoUtils.closeQuietly(serverPipeFd);
                serverPipeFd = null;
                //处理子进程参数 返回command 关闭server socket
                return handleChildProc(parsedArgs, childPipeFd, parsedArgs.mStartChildZygote);
            }
        ...    
 }
```
通过调用`handleChildProc`处理子进程-应用程序进程。关于应用程序进程的启动的详细过程后面单独篇幅详细讲解。
```cpp
    private Runnable handleChildProc(ZygoteArguments parsedArgs,
            FileDescriptor pipeFd, boolean isZygote) {
       
        //因此应用程序进程是zygote fork过来了，所以也拥有ZygoteServer但是应用程序进程不需要
        //socket通信 所以进行了关闭 避免资源浪费
        closeSocket();
        ...       
        //后面的代码在 后续文章中详细讲解
            if (!isZygote) {
                return ZygoteInit.zygoteInit(parsedArgs.mTargetSdkVersion,
                        parsedArgs.mDisabledCompatChanges,
                        parsedArgs.mRemainingArgs, null /* classLoader */);
            } else {
                return ZygoteInit.childZygoteInit(parsedArgs.mTargetSdkVersion,
                        parsedArgs.mRemainingArgs, null /* classLoader */);
            }
        }
    }
```

#### socket 监听的流程

1. 通过`Os.poll`对`mZygoteSocket`的`FD`监听`POLLIN`事件（此时`socketFDs`列表里面只有`mZygoteSocket`对象）
2. 当另一个进程A通过`ZygoteProcess.connect`方法，来创建一个连接并发出消息（此时调用了`Os.scoket`，创建`Socket`连接）
3. `Os.poll`触发，此时一定触发`if(pollIndex == 0)`的条件，然后创建一个新的`Zygote`连接（调用了`Os.accept`）
4. 进程A通过在`Zygote.connect`这个方法流程中，调用`LocalSocketImpl.connectLocal`方法，确认连接
5. 然后`Os.poll`第二次监听到`POLLIN`事件，`socketFDs`现在里面有了`mZygoteSocket`和`新连接返回的文件描述符`
6. 此时会触发`pollIndex < usapPoolEventFDIndex`,去`fork`一个新的进程，并找到对应的`main`方法，子进程创建完毕
> PS: 这里看不懂没关系，在后续的Android 应用进程启动原理中 进行详细讲解，这里先做了解。

### 总结

1. 调用AndroidRuntime的start函数，启动Zygote进程
2. 创建Java虚拟机并为Java虚拟机注册JNI方法
3. 通过JNI调用ZygoteInit的main方法进入Zygote的Java框架层
4. 创建服务端Socket，并通过runSelectLoop方法等待AMS的请求来创建新的应用程序进程
5. 启动SystemServer进程

![](./Zygote%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8%E5%88%86%E6%9E%90.assets/1684491900297-3c8d5334-1a8c-4e56-96a8-3516e4a2dcc8.jpeg)