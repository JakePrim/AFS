# Init 进程启动过程

> Framework 源码剖析系列 : 源码依据 Android 11 系统,为什么看 Android11 的源码？因为 Android11 源码比之前的 Android 8 等源码有很大部分的改动，目前 Android 已经到了 14 版本。

在学习 Init 进程之前，需要理解两个在Framework非常常见的函数`fork`函数和`exec`函数族。

在framework层经常会用到`fork`函数，`fork`函数从字面意思上看就是 `fork` 拷贝一个进程。

`fork`函数通过系统调用，创建一个与原来进程几乎完全相同的进程，也就是两个进程可以做完全相同的事，但如果**初始参数**或者**传入的变量**不同，两个进程也可以做不同的事。

`fork`函数调用后，创建的新进程叫做**子进程**，原来存在的进程叫做**父进程**。

**exec 函数族**：在fork后的子进程中使用exec函数族，可以装入和运行其他程序，`exec`函数族共有6种不同形式的函数：**execl**  **execle**  **execlp**  **execv**  **execve**  **execvp**. `exec` 函数族**可以根据指定的文件名或目录名找到可执行文件**，并用它来取代原来调用进程的**数据段、代码段和堆栈段**。**在执行完后，原调用进程的内容除了进程号PID外，其他全部被新程序的内容替换了**。

例如：init进程fork了一个子进程，通过exec函数族加载zygote文件，那么这个子进程就变成了zygote进程。

### init进程

init 进程是Android系统中**用户空间**的第一个进程，作为第一个进程，它被赋予了很多极其重要的工作职责，例如创建zygote孵化器和属性服务等。init进程是由多个源文件共同组成的，这些文件位于源码目录的`system/core/init`

Android 系统启动流程：

1. 启动电源以及系统启动：当电源按下时引导芯片代码开始从预定义的地方（固化在ROM）开始执行，加载引导程序`Bootloader` 到RAM，然后执行
2. 引导程序 `boot loader`：引导程序是在Android操作系统运行前的一个小程序，它的主要作用是把系统OS拉起来并运行，并初始化一些硬件参数等功能。
3. Linux内核启动：内核启动后，主要启动了两个进程，分别是`swapper` 进程 `pid = 0`,`kthreadd`进程 pid=2.`swapper`进程又被成为`idle`进程，系统初始化过程Kernel由无到有开创的第一个进程，**用于初始化进程管理、内存管理、会加载硬件屏幕、相机硬件等**。`kthreadd`进程是**内核进程**，会创建内核中的一些线程，以及内核守护进程，这个进程是内核进程的鼻祖。Android平台的基础是Linux内核，比如ART虚拟机最终调用底层Linux内核来执行功能。Linux内核的安全机制为Android提供相应的保障，也允许设备制造商为内核开发硬件驱动程序。

4. 启动Kernel的swapper进程(pid=0)：该进程又称为idle进程, 系统初始化过程Kernel由无到有开创的第一个进程, 用于初始化进程管理、内存管理，加载Display,Camera Driver，Binder Driver等相关工作；

5. 启动kthreadd进程（pid=2）：是Linux系统的内核进程，会创建内核工作线程kworkder，软中断线程ksoftirqd，thermal等内核守护进程。kthreadd进程是所有内核进程的鼻祖。

6. Init进程启动：Linux内核启动后，会在用户空间启动第一个进程（第一个用户进程），即init进程 pid = 1,**init进程是所用用户进程的鼻祖**，init进程初始化和启动属性服务，并且fork zygote进程。init进程会孵化出ueventd、logd、healthd、installd、adbd、lmkd等用户守护进程；init进程还启动`ServiceManager`(binder服务管家)、bootanim(开机动画)等重要服务；init进程孵化出Zygote进程，Zygote进程是Android系统的第一个**Java进程**(即虚拟机进程)，**Zygote是所有Java进程的父进程**，Zygote进程本身是由init进程孵化而来的。

7. Zygote进程启动：创建Java虚拟机JVM，创建服务端socket，fork `SystemServer`进程。是由init进程通过解析init.rc文件后fork生成的，Media Server进程，是由init进程fork而来，负责启动和管理整个C++ framework，包含AudioFlinger，Camera Service等服务。

8. SystemServer 进程启动：**启动Binder线程池**和**SystemServiceManager**，并且启动各种系统服务。**System Server负责启动和管理整个Java framework**，包含ActivityManager，WindowManager，PackageManager，PowerManager等服务

9. Launcher 启动：被SystemServer进程启动的**ActivityManagerService** AMS 会启动Launcher，Launcher启动后会将已安装应用的快捷图标显示到界面上。Zygote进程孵化出的第一个App进程是Launcher，这是用户看到的桌面App，**所有的APP进程都是由zygote进程孵化而来的**。

![img](https://cdn.nlark.com/yuque/0/2022/png/375694/1657790530902-6730c59e-9b29-49e2-bf8d-6b9e1e5aeb83.png)

下面来看 Init 进程在启动过程中主要做了什么？Android 7.0 和 Android 11 在 init 进程的入口函数有很大的变化，但是主要做的核心方法没有太大的变化。

如下图是Android 7.0 的源码：主要做了

1. 文件的创建和挂载
2. 启动属性服务
3. 解析init.rc配置文件

![img](https://cdn.nlark.com/yuque/0/2022/png/375694/1657784013899-0246a074-33d3-45e1-ac8c-0ff87249948c.png)

在Android11中的源码存在差异：在Android11的源码中` init.cpp`中没有了`main`函数，而`main`函数放到了`main.cpp`中。`main`函数会被**启动多次(通过exec函数族来进行启动多次也是文章开篇所说的)**,会根据传递的不同参数，调用不同的方法。为什么要调用多次呢？

代码如下所示：

```cpp
int main(int argc, char** argv) {
#if __has_feature(address_sanitizer)
    __asan_set_error_report_callback(AsanReportCallback);
#endif

    //strcmp 比对字符串的函数
    if (!strcmp(basename(argv[0]), "ueventd")) {
        return ueventd_main(argc, argv);
    }

    if (argc > 1) {
        if (!strcmp(argv[1], "subcontext")) {
            android::base::InitLogging(argv, &android::base::KernelLogger);
            const BuiltinFunctionMap& function_map = GetBuiltinFunctionMap();

            return SubcontextMain(argc, argv, &function_map);
        }
    	//2 -> SetupSelinux在FirstStageMain执行后执行
        if (!strcmp(argv[1], "selinux_setup")) {
            return SetupSelinux(argv);
        }
    	//3 -> 在SetupSelinux执行后执行
        if (!strcmp(argv[1], "second_stage")) {
            return SecondStageMain(argc, argv);
        }
    }
    //1 -> 先执行这个函数
    return FirstStageMain(argc, argv);
}
```

首先来看`FirstStageMain`做了些什么？

1. 创建和挂载文件操作
2. 初始化Log系统
3. execv 函数将init进程的内容除进程号外，由新的程序替换`SetupSelinux`

通过`execv`又会执行`main.cpp`的`main`函数，参数`[system/bin/init,"selinux_setup",nullptr]`就会执行`SetupSelinux`函数

```cpp
int FirstStageMain(int argc, char** argv) {
    ...
	//以上代码 基本是挂载文件系统、创建文件等操作
    //把标准输入、标准输出、标准错误重定向到 /dev/null
    SetStdioToDevNull(argv);
    // Now that tmpfs is mounted on /dev and we have /dev/kmsg, we can actually
    // talk to the outside world...
    //初始化本阶段的内核日志
    InitKernelLogging(argv);
    ...
    const char* path = "/system/bin/init";
    const char* args[] = {path, "selinux_setup", nullptr};
    auto fd = open("/dev/kmsg", O_WRONLY | O_CLOEXEC);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);
    close(fd);
    //装入并运行其他程序的函数 再次运行main函数 传递参数[system/bin/init,"selinux_setup",nullptr]
    execv(path, const_cast<char**>(args));

    // execv() only returns if an error happened, in which case we
    // panic and never fall through this conditional.
    PLOG(FATAL) << "execv(\"" << path << "\") failed";
    return 1;
}
```

上述代码执行完毕会执行到`SetupSelinux`函数

```cpp
 //参数比对：[system/bin/init,"selinux_setup",nullptr]，[1] -> selinux_setup
        if (!strcmp(argv[1], "selinux_setup")) {
            //selinux.cpp -> 2
            return SetupSelinux(argv);
        }
```

`SetupSelinux` 函数代码如下所示：

主要工作是初始化`selinux`、加载 `selinux` 规则。`selinux` 是 `Linux` 中的一个安全控制模块，在这种访问控制体系的限制下， 进程只能访问那些在他的任务中所需要的文件。在 `SetupSelinux` 函数执行结束的时候同样通过 `execv` 命令启动了 `Init` 的第三个过程 `SecondStageMain`。

```cpp
int SetupSelinux(char** argv) {
	...
    const char* path = "/system/bin/init";
    const char* args[] = {path, "second_stage", nullptr};
    //再次调用main函数 传递参数second_stage
    execv(path, const_cast<char**>(args));
}
```

最终会执行`SecondStageMain`函数:

1. 初始化并启动属性服务
2. 初始化子进程终止处理函数
3. 解析init.rc文件
4. 循环等待新的Action到来,调用Action的处理函数

```cpp
int SecondStageMain(int argc, char** argv) {
	...
    //系统属性初始化
    PropertyInit();
    ...
    //创建Epoll,epoll是Linux内核可扩展的I/O事件通知机制
    Epoll epoll;
    if (auto result = epoll.Open(); !result.ok()) {
        PLOG(FATAL) << result.error();
    }
    //注册信号处理
    InstallSignalFdHandler(&epoll);
    //加载默认的系统属性
    InstallInitNotifier(&epoll);
    //启动属性服务 system/core/init/property_service.cpp
    StartPropertyService(&property_fd);
    ...
    const BuiltinFunctionMap& function_map = GetBuiltinFunctionMap();
    //Action对应的执行方法
    Action::set_function_map(&function_map);
        if (!SetupMountNamespaces()) {
        PLOG(FATAL) << "SetupMountNamespaces failed";
    }

    InitializeSubcontext();

    ActionManager& am = ActionManager::GetInstance();
    ServiceList& sm = ServiceList::GetInstance();
    //解析init.rc文件
    LoadBootScripts(am, sm);
    ...
    while (true) {
        ...
        //actions的触发
        if (!(prop_waiter_state.MightBeWaiting() || Service::is_exec_service_running())) {
            am.ExecuteOneCommand();
        }
        ...
    }
    ...
}
```

`SecondStageMain`是核心的代码逻辑，首先先看init.rc文件是什么、以及如何解析init.rc文件的。

### init.rc 文件

init.rc 是一个配置文件，内部由`Android初始化语言`编写 Android Init Language 编写的脚本，主要包含5种类型语句：

Actions（行为） 、 Commands（命令） 、 Services （服务）、Options （选项）、Imports

init.rc 只是一个语法配置文件就像一个xml文件一样，没有执行顺序的，解析器通过读这个文件获取想要的数据，包括service，action等

Action 类型语句的格式如下：拥有一个trigger来确定合适执行这些命令

```cpp
on <trigger> [&& <trigger>]*  //设置触发器
    <command>
    <command> //动作触发之后要执行的命令
```

Services 类型语句格式如下：

option 用于指定何时和怎样启动service，在Actions中有一条命令`class_start <服务类别名> `用于启动所有未运行的相同类别的service，而option可以通过`class 服务类别名`对service的类别名进行指定。

**service的启动一般是通过Actions的class_start命令进行启动**

```cpp
service <name> <pathname> [<argument>]* //<service的名字><执行程序路径><传递参数>
    <option> //option是service的修饰词，影响什么时候、如何启动services
    <option>
```

注意在Android 7.0之后：对init.rc文件进行了拆分、每个service都是一个rc文件。

更多的rc的资料参考链接：

[Android 启动过程简析（一）之 init 进程 - 掘金](https://juejin.cn/post/6844903506915098637)

[Android系统init.rc全解析](https://mazhidong.github.io/post/aosp/2019-01-18_android系统init.rc全解析/)

以如下rc文件示例：zygote service 启动zygote进程，`class name:main`

`service zygote`:用于通知init进程创建zygote进程

`/system/bin/app_process64`：zygote进程执行程序的路径

`-Xzygote /system/bin --zygote --start-system-server`：要传给`app_process64`的参数

```
class main`：zygote的`class name`为`main
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote stream 660 root system
    socket usap_pool_primary stream 660 root system
    onrestart exec_background - system system -- /system/bin/vdc volume abort_fuse
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks
```

那么问题来了：

1. 如何解析init.rc文件？
2. `class_start`对应的处理函数是什么？
3. `nonencrypted` 什么时候被触发？
4. init进程是如何创建`zygote`进程呢？

从init.rc文件中可以找到如下脚本：当`nonencrypted`这个触发器启动的时候就会启动`class_start main`那么就会启动zygote serivce.

```cpp
on nonencrypted
    class_start main
    class_start late_start
```

### 解析init.rc
回到`init.cpp`中`SecondStageMain`�方法中`LoadBootScripts`解析`init.rc`文件
```cpp
    const BuiltinFunctionMap& function_map = GetBuiltinFunctionMap();
    //Action对应的执行方法 保存在一个静态变量中
    Action::set_function_map(&function_map);
    ....
    ActionManager& am = ActionManager::GetInstance();
    ServiceList& sm = ServiceList::GetInstance();
    //解析init.rc
    LoadBootScripts(am, sm);
```
`LoadBootScripts`解析init.rc,交给`Parser`对象去解析调用`ParseConfig`方法，`set_function_map` 设置对应的处理函数，这个后面在看。
```cpp
static void LoadBootScripts(ActionManager& action_manager, ServiceList& service_list) {
    //创建一个解析器
    Parser parser = CreateParser(action_manager, service_list);
    //获取ro.boot.init_rc属性的值
    std::string bootscript = GetProperty("ro.boot.init_rc", "");
    if (bootscript.empty()) {
        parser.ParseConfig("/system/etc/init/hw/init.rc");
        if (!parser.ParseConfig("/system/etc/init")) {
            late_import_paths.emplace_back("/system/etc/init");
        }
        // late_import is available only in Q and earlier release. As we don't
        // have system_ext in those versions, skip late_import for system_ext.
        parser.ParseConfig("/system_ext/etc/init");
        if (!parser.ParseConfig("/product/etc/init")) {
            late_import_paths.emplace_back("/product/etc/init");
        }
        if (!parser.ParseConfig("/odm/etc/init")) {
            late_import_paths.emplace_back("/odm/etc/init");
        }
        if (!parser.ParseConfig("/vendor/etc/init")) {
            late_import_paths.emplace_back("/vendor/etc/init");
        }
    } else {
        parser.ParseConfig(bootscript);
    }
}
```
创建一个解析器：添加了三个section分别是:service、on、import，`ServiceParser` 解析service、`ActionParser`解析action、`ImportParser` 解析import，通过调用`AddSectionParser`添加对应的解析器到`section_parsers_`�的Map集合中。
```cpp

Parser CreateParser(ActionManager& action_manager, ServiceList& service_list) {
    Parser parser;

    parser.AddSectionParser("service", std::make_unique<ServiceParser>(
                                               &service_list, GetSubcontext(), std::nullopt));
    parser.AddSectionParser("on", std::make_unique<ActionParser>(&action_manager, GetSubcontext()));
    parser.AddSectionParser("import", std::make_unique<ImportParser>(&parser));

    return parser;
}
void Parser::AddSectionParser(const std::string& name, std::unique_ptr<SectionParser> parser) {
    section_parsers_[name] = std::move(parser);
}
std::map<std::string, std::unique_ptr<SectionParser>> section_parsers_;
ParseConfig` 方法如下对path进行解析，如果是Dir则会递归调用，最终会调用`ParseConfigFile
bool Parser::ParseConfig(const std::string& path) {
    if (is_dir(path.c_str())) {
        return ParseConfigDir(path);
    }
    return ParseConfigFile(path);
}
```
`ParseConfigFile` 会调用到`ParseData`解析读取到的内容数据
```cpp
bool Parser::ParseConfigFile(const std::string& path) {
    LOG(INFO) << "Parsing file " << path << "...";
    android::base::Timer t;
    //从rc文件读取内容 保存config_contents中
    auto config_contents = ReadFile(path);
    if (!config_contents.ok()) {
        LOG(INFO) << "Unable to read config file '" << path << "': " << config_contents.error();
        return false;
    }
    //解析读取到的内容
    ParseData(path, &config_contents.value());

    LOG(VERBOSE) << "(Parsing " << path << " took " << t << ".)";
    return true;
}
```
�ParseData解析的核心代码如下：
> 如果是 T_TEXT 则会保存在 args 中，如果是 T_NEWLINE 则会交由对应的解析器进行解析

```cpp
void Parser::ParseData(const std::string& filename, std::string* data) {
...
//当前对应的解析器    
SectionParser* section_parser = nullptr;    
...
//如果上一次存在解析，则解析结束
auto end_section = [&] {
        bad_section_found = false;
        if (section_parser == nullptr) return;
        //执行解析器的EndSection函数    
        if (auto result = section_parser->EndSection(); !result.ok()) {
            parse_error_count_++;
            LOG(ERROR) << filename << ": " << section_start_line << ": " << result.error();
        }
        section_parser = nullptr;
        section_start_line = -1;
    };
...    
case T_NEWLINE: {
                state.line++;
                //如果args为空则不解析
                if (args.empty()) break;
                ....
                } else if (section_parsers_.count(args[0])) {
                    //判断是不是一个section的起始位置，是不是on service import其中一个
                    end_section();
                    //取出对应的解析器section_parsers_ 是在AddSectionParser函数
                    //进行设置的而AddSectionParser函数是在CreateParser函数里调用
                    section_parser = section_parsers_[args[0]].get();
                    section_start_line = state.line;
                    //进行section解析ParseSection
                    if (auto result =
                                section_parser->ParseSection(std::move(args), filename, state.line);
                        !result.ok()) {
                        parse_error_count_++;
                        LOG(ERROR) << filename << ": " << state.line << ": " << result.error();
                        section_parser = nullptr;
                        bad_section_found = true;
                    }
                } else if (section_parser) {//不是则说明是section的子块，进行Line解析
                    if (auto result = section_parser->ParseLineSection(std::move(args), state.line);
                        !result.ok()) {
                        parse_error_count_++;
                        LOG(ERROR) << filename << ": " << state.line << ": " << result.error();
                    }
                } 
        ...
    //  解析完成后清空
                args.clear();
                break;
            }
...
}
```
但这里依旧没有找到我们想要的答案，从上述代码中可以看到调用了`ParseSection`�、`ParseLineSection`、`EndSection`�进行解析，其实是从`section_parsers_`中获取到对应的解析器实例，看`ActionParse`中是如何解析的.
### ActionParse -> 解析Action
`ParseSection` 方法主要是创建`Action`对象，添加触发器
```cpp
Result<void> ActionParser::ParseSection(std::vector<std::string>&& args,
                                        const std::string& filename, int line) {
    ...

    //添加触发器
    if (auto result =
                ParseTriggers(triggers, action_subcontext, &event_trigger, &property_triggers);
        !result.ok()) {
        return Error() << "ParseTriggers() failed: " << result.error();
    }
    auto action = std::make_unique<Action>(false, action_subcontext, filename, line, event_trigger,
                                           property_triggers);
    //将 aciton_ 指针移动到当前的 action
    action_ = std::move(action);
    return {};
}
```
`ParseLineSection` 查找对应的command处理函数，command类似class_start等
```cpp
Result<void> ActionParser::ParseLineSection(std::vector<std::string>&& args, int line) {
    //将解析的command增加到当前action的commands_中   
    return action_ ? action_->AddCommand(std::move(args), line) : Result<void>{};
}
//commands_ 在 action.h 中的定义
std::vector<Command> commands_;
```
如下代码给command添加处理函数，从`function_map_`中获取处理函数
```cpp
Result<void> Action::AddCommand(std::vector<std::string>&& args, int line) {
    //如果处理函数不存在 则抛出异常
    if (!function_map_) {
        return Error() << "no function map available";
    }
    //为command添加处理函数
    auto map_result = function_map_->Find(args);
    if (!map_result.ok()) {
        return Error() << map_result.error();
    }

    commands_.emplace_back(map_result->function, map_result->run_in_subcontext, std::move(args),
                           line);
    return {};
}
```
`EndSection` 将解析完成后的action添加到`ActionManager`的 `actions_`中
```cpp
Result<void> ActionParser::EndSection() {
    if (action_ && action_->NumCommands() > 0) {
        action_manager_->AddAction(std::move(action_));
    }

    return {};
}

//action_manager.cpp
void ActionManager::AddAction(std::unique_ptr<Action> action) {
    actions_.emplace_back(std::move(action));
}
```
在action.h中`set_function_map` 这个方法其实就是保存要执行解析action后的方法
```cpp
class Action {
  public:
    ...
    static void set_function_map(const BuiltinFunctionMap* function_map) {
        function_map_ = function_map;
    }

  private:
    ....
    static const BuiltinFunctionMap* function_map_;
};
```
在`system/core/init/builtins.cpp`中找到了`GetBuiltinFunctionMap`�方法，找到了rc文件中对应的执行函数`class_start -> do_class_start`
![image.png](https://cdn.nlark.com/yuque/0/2023/png/375694/1684041334930-c6807935-b287-4042-968e-6c0ac9c90bdd.png#averageHue=%231e1d1d&clientId=ufc29bc9b-71e8-4&from=paste&height=659&id=u01a7e6e4&originHeight=1318&originWidth=1686&originalType=binary&ratio=2&rotation=0&showTitle=false&size=333183&status=done&style=stroke&taskId=u4fdaabb2-b11f-4597-b8e5-b1c2cb1bbc2&title=&width=843)
如class_start的方法调用，通过`ServcieList` 去匹配`classnames()` ，arg[1]是传递的参数：`main`

```cpp
static Result<void> do_class_start(const BuiltinArguments& args) {
    // Do not start a class if it has a property persist.dont_start_class.CLASS set to 1.
    if (android::base::GetBoolProperty("persist.init.dont_start_class." + args[1], false))
        return {};
    //查找对应的class name
    for (const auto& service : ServiceList::GetInstance()) {
        if (service->classnames().count(args[1])) {
            if (auto result = service->StartIfNotDisabled(); !result.ok()) {
                LOG(ERROR) << "Could not start service '" << service->name()
                           << "' as part of class '" << args[1] << "': " << result.error();
            }
        }
    }
    return {};
}
```
> `ActionParse`的整体流程：
> 实际上是创建一个 Action 对象，然后为 Action 对象添加 Trigger 以及对应的 Command，其中在添加 Command 的过程中还为 Command 指定了处理函数，最后在将 Action 对象增加到 ActionManager vector 类型的 actions_ 链表当中去。

### ServiceParse -> 解析service
Service Parse解析过程和Action Parse解析过程大致一样。

- ParseSection 创建service对象，将service_移动到当前到Service对象
- ParseLineSection为service的每个option添加处理函数
- EndSection 的主要工作是将解析完成的 service （域填充完毕的 Service 对象）添加到 ServiceManager 的 services_ 中
```cpp
    Result<void> ParseSection(std::vector<std::string>&& args, const std::string& filename,
                              int line) override;
    Result<void> ParseLineSection(std::vector<std::string>&& args, int line) override;
    Result<void> EndSection() override;
```

通过上述代码分析找到了class_start的对应函数。包括action的command的对应处理函数，service的option的对应处理函数。
### Actions如何触发
继续回到init.cpp,command的执行是通过`ActionManager`的`ExecuteOneCommand`函数触发，在SecondStageMain 函数中while(true) 进入到一个无限循环等待状态
```cpp
...
while (true) {
        // By default, sleep until something happens.
        auto epoll_timeout = std::optional<std::chrono::milliseconds>{};

        auto shutdown_command = shutdown_state.CheckShutdown();
        if (shutdown_command) {
            HandlePowerctlMessage(*shutdown_command);
        }
        //actions的触发 如果有要处理的action执行对应的处理函数
        if (!(prop_waiter_state.MightBeWaiting() || Service::is_exec_service_running())) {
            am.ExecuteOneCommand();
        }
    ...
}
....
```
而`ActionManager.ExecuteOneCommand`调用了`Action.ExecuteOneCommand`
```cpp
void ActionManager::ExecuteOneCommand() {
    ...   
    action->ExecuteOneCommand(current_command_);
    ...
}

void Action::ExecuteOneCommand(std::size_t command) const {
    // We need a copy here since some Command execution may result in
    // changing commands_ vector by importing .rc files through parser
    Command cmd = commands_[command];
    ExecuteCommand(cmd);
}


void Action::ExecuteCommand(const Command& command) const {
    ...
    //调用处理函数
    auto result = command.InvokeFunc(subcontext_);
    ....   
}
```
其实上述代码很简单，在init进程会开启一个while(true)无限循环，然后从`ActionManager`的`action_`中保存的`action`顺序依次对每个`Action`进行处理，而在这个过程中`system/core/init/builtins.cpp`中会执行`do_class_start`函数，调用了`Service`的`StartIfNotDisabled`函数
```cpp
Result<void> Service::StartIfNotDisabled() {
    if (!(flags_ & SVC_DISABLED)) {
        return Start();
    } else {
        flags_ |= SVC_DISABLED_START;
    }
    return {};
}
```
最终调用了`Service`的`start`函数，创建zygote进程.
```cpp
Result<void> Service::Start() {
    ....
    //创建子进程
    pid_t pid = -1;
    if (namespaces_.flags) {
        pid = clone(nullptr, nullptr, namespaces_.flags | SIGCHLD, nullptr);
    } else {
        pid = fork();
    }  

}
```
在`frameworks/base/cmds/app_process/app_main.cpp`中zygote最终在main函数中被启动.
```cpp
int main(int argc, char* const argv[])
{
    ...
    if (zygote) {
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
> init进程是系统中第一个用户进程，主要作用是创建用户空间 文件夹并挂在、启动属性服务、解析init.rc文件并启动zygote进程。
> init启动zygote的过程：init进程通过解析init.rc文件将action保存在ActionManager的action_链表中，然后通遍历actions_链表，执行action命令对应的处理函数，从而转至builtins.cpp的do_class_start函数，之后通过Service的StartIfNotDisabled调用Service的Start函数，最终通过Start函数创建zygote进程，执行对应的app_main.cpp文件启动zygote.

Zygote的具体的启动过程我会在下一篇文章中详细讲解。
### 属性服务
> Windows 平台上有一个注册表管理器，注册表的内容采用键值对的形式来记录用户、软件的一些使用信息。即使系统或者软件重启，它还是能够根据之前在注册表中的记录，进行相应的初始化工作。
> Android 也提供了一个类似的机制，叫做属性服务。

如下代码所示：调用了`StartPropertyService`启动属性服务
```cpp
...
//启动属性服务 system/core/init/property_service.cpp
    StartPropertyService(&property_fd);
...
```
属性服务启动的核心代码如下所示：
创建了一个socket server服务，init进行监听到socket信息就会处理
```cpp
void StartPropertyService(int* epoll_socket) {
    InitPropertySet("ro.property_service.version", "2");

    int sockets[2];
    if (socketpair(AF_UNIX, SOCK_SEQPACKET | SOCK_CLOEXEC, 0, sockets) != 0) {
        PLOG(FATAL) << "Failed to socketpair() between property_service and init";
    }
    *epoll_socket = from_init_socket = sockets[0];
    init_socket = sockets[1];
    StartSendingMessages();
    //CreateSocket 创建一个非阻塞的socket，auto 自动推断变量类型
    if (auto result = CreateSocket(PROP_SERVICE_NAME, SOCK_STREAM | SOCK_CLOEXEC | SOCK_NONBLOCK,
                                   false, 0666, 0, 0, {});
        result.ok()) {
        property_set_fd = *result;//result 所指向的对象 -> socket
    } else {
        LOG(FATAL) << "start_property_service socket creation failed: " << result.error();
    }

    //监听socket -> property_set_fd,这样创建的socket就成了server属性服务，8 可以同时为8个视图设置属性的用户提供服务
    listen(property_set_fd, 8);
    //创建了一个线程 PropertyServiceThread
    auto new_thread = std::thread{PropertyServiceThread};
    property_service_thread.swap(new_thread);
}
```
`PropertyServiceThread` 注册一个处理函数，当socket有数据到来时，调用`handle_property_set_fd`函数处理
```cpp
static void PropertyServiceThread() {
    Epoll epoll;
    if (auto result = epoll.Open(); !result.ok()) {
        LOG(FATAL) << result.error();
    }
    //将property_set_fd放入到epoll的句柄中，监听property_set_fd 当socket有数据到来时init进程会调用handle_property_set_fd
    //进行处理
    if (auto result = epoll.RegisterHandler(property_set_fd, handle_property_set_fd);
        !result.ok()) {
        LOG(FATAL) << result.error();
    }

    if (auto result = epoll.RegisterHandler(init_socket, HandleInitSocket); !result.ok()) {
        LOG(FATAL) << result.error();
    }

    while (true) {
        auto pending_functions = epoll.Wait(std::nullopt);
        if (!pending_functions.ok()) {
            LOG(ERROR) << pending_functions.error();
        } else {
            for (const auto& function : *pending_functions) {
                (*function)();
            }
        }
    }
}
```

init进程启动时做了很多工作，主要做了三件事：

1. 创建和挂载启动所需的文件目录
2. 初始化和启动属性服务
3. 解析init.rc配置文件并启动Zygote进程

![](./Init%E8%BF%9B%E7%A8%8B%E5%90%AF%E5%8A%A8%E5%88%86%E6%9E%90.assets/1685414338325-3de40863-8979-46c4-bc08-eb24f50c8ab4.jpeg)