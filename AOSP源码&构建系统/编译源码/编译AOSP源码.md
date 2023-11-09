# 编译AOSP源码

## 硬件环境

AOSP 的下载、编译理论上支持 Mac和Linux，但是不建议在非 Linux 环境下尝试编译，其中要踩的坑实在太多。

下载 Android 12 及以后的版本，需要至少100GB的硬盘空间，编译Android 12 则需要300GB的硬盘空间和16GB以上的内存，低于这个配置在编译时大概率会报错。我的硬件环境如下：Ubuntu 20.0.4版本的系统

![image-20231103134507171](./%E7%BC%96%E8%AF%91AOSP%E6%BA%90%E7%A0%81.assets/image-20231103134507171.png)

## 源码管理工具

安装完Ubuntu系统后，在下载Android源码之前，要先安装其构建工具 Repo 来初始化源码。Repo是Android用来辅助Git工作的一个工具。(Repo命令的详解在下一篇文章中讲解)

### Repo

#### Repo 简介

Repo是Google使用python脚本编写的用于调用Git的脚本，主要用来下载、管理Android项目的软件仓库。

Repo不会取代Git，但是它可以让开发者在Android环境中更加轻松的使用Git。

在大多数情况下，确实可以仅使用 Git（不必使用 Repo），或结合使用 Repo 和 Git 命令以组成复杂的命令。不过，使用 Repo 执行基本的跨网络操作可大大简化我们的工作。

#### 安装 Repo

Android源码同时使用git和repo进行管理，repo是基于git的代码管理工具，类似github、gitee，所以需要同时安装git和repo

```
sudo apt-get update
sudo apt-get install repo
```

如果没有~/bin/repo目录，需要先创建一下，安装好后验证是否安装成功

```
repo verison
```

如果出现

```
<repo not installed>
repo launcher version 2.15
(from /usr/bin/repo)
```

说明安装成功了，repo安装好后可以开始下载源码了

第 1 步，创建一个bin目录，并将这个目录添加到系统的环境变量中

```bash
mkdir ~/bin
PATH=~/bin:$PATH
```

第 2 步，下载 repo

```bash
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

安装完repo后需要使用`source`指令重新加载linux的配置环境，这里我们简单一些直接重启操作系统就行。



### Git

#### Git 简介

Git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。也是目前使用范围最广的版本控制系统。

Android 使用 Git 执行本地操作，例如建立本地分支、提交、对比差异、修改。Google 最初决定使用一种分布式修订版本控制系统，经过筛选最后选中了Git。

> 对于分支要求不高的项目会采用SVN或其它在线办公软件管理。例如，在UP的公司，车载项目的文档类资料以前采用SVN进行管理，现在已经统一切换到飞书系统上。

#### 安装 Git

第 1 步，执行安装指令

```
sudo apt install git
```

第 2 步，配置 Git 全局环境

```arduino
git config --global user.name "用户名"
git config --global user.email "邮箱"
```

第 3 步， 生成 ssh 秘钥（可选配置）

```
ssh-keygen
```

一直按回车，生成的秘钥文件会在 `~/.ssh`目录下载，我们将`~/.ssh`目录下的`id_rsa.pub` 里面的内容复制到仓库管理系统相应的SSH Key中，在后续的代码下载时就不需要再输入用户名和密码。

> 车载项目常见的源码仓库管理系统是Gerrit 和 GItLab。

### 在线阅读、检索AOSP源码

[aospxref.com/](https://link.juejin.cn?target=http%3A%2F%2Faospxref.com%2F) 是一个在线的 AOSP 源码阅读网站，在这个网站上我们可以很轻松，阅读、检索 Android 的源码，而且对于车载 Android 开发而言，阅读源码十分的重要。

### Git

#### Git 简介

Git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。也是目前使用范围最广的版本控制系统。

Android 使用 Git 执行本地操作，例如建立本地分支、提交、对比差异、修改。Google 最初决定使用一种分布式修订版本控制系统，经过筛选最后选中了Git。

> 对于分支要求不高的项目会采用SVN或其它在线办公软件管理。例如，在UP的公司，车载项目的文档类资料以前采用SVN进行管理，现在已经统一切换到飞书系统上。

#### 安装 Git

第 1 步，执行安装指令

```
sudo apt install git
```

第 2 步，配置 Git 全局环境

```arduino
git config --global user.name "用户名"
git config --global user.email "邮箱"
```

第 3 步， 生成 ssh 秘钥（可选配置）

```
ssh-keygen
```

一直按回车，生成的秘钥文件会在 `~/.ssh`目录下载，我们将`~/.ssh`目录下的`id_rsa.pub` 里面的内容复制到仓库管理系统相应的SSH Key中，在后续的代码下载时就不需要再输入用户名和密码。

> 车载项目常见的源码仓库管理系统是Gerrit 和 GItLab。

### 在线阅读、检索AOSP源码

[aospxref.com/](https://link.juejin.cn?target=http%3A%2F%2Faospxref.com%2F) 是一个在线的 AOSP 源码阅读网站，在这个网站上我们可以很轻松，阅读、检索 Android 的源码，而且对于车载 Android 开发而言，阅读源码十分的重要。

## 下载 AOSP



1. ### 下载方式

下载 AOSP 有以下两种方式

- 使用初始化源码包

中科大和清华大学提供了AOSP源码的压缩包，现在的包大约有60GB左右，可以使用命令行或迅雷等下载工具下载。

优点：下载速度快，支持断点续传。

缺点：不贴近工作环境，实际项目中不使用这种方式。

清华大学文档：[mirrors.tuna.tsinghua.edu.cn/help/AOSP/](https://link.juejin.cn?target=https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fhelp%2FAOSP%2F)

中科大文档：[mirrors.ustc.edu.cn/help/aosp.h…](https://link.juejin.cn?target=https%3A%2F%2Fmirrors.ustc.edu.cn%2Fhelp%2Faosp.html)

- 使用 repo 直接同步源码

使用repo直接同步源码，是Android官方文档中使用的方式。

优点：更贴近工作环境。在实际项目中也是使用这种方式同步Android源码。

缺点：源码太大，外网不稳定、速度很慢，容易同步失败（实际项目中，我们使用公司内网不会存在这个问题）。

官方文档：[source.android.google.cn/docs/setup/…](https://link.juejin.cn?target=https%3A%2F%2Fsource.android.google.cn%2Fdocs%2Fsetup%2Fdownload%2Fdownloading%3Fhl%3Dzh-cn)

由于下载Android源码非常消耗服务器的带宽和I/O，中科大和清华大学的文档中都强烈要求我们使用第一种方式同步源代码。

如果下载Android官方的AOSP的源码，UP建议使用下载压缩包的方式，如果是下载公司项目的Android源码，则只能使用repo同步。

下面我们分别演示这两种下载Android源码的方式：

#### 第 1 种方式，使用Android源码包

**第 1 步，建立工作目录**

开始下载AOSP源码之前，需要在合适的位置创建一个目录用于放置源码，使用如下命令

```bash
mkdir AOSP
cd AOSP
```

**第 2 步，下载AOSP初始化包**

下载AOSP初始化也有两种方式

第一种，是使用CURL命令行工具。个人建议使用这种方式下载，如果中途意外中断，在执行一次同样的CURL指令，即可进行断点续传。

```arduino
curl -OC - https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar
```

第二种，使用下载工具下载源码包。下载工具可以是浏览器，也可以是迅雷等专用下载工具

下载初始化包，可以用任意操作系统，甚至只要你的手机闪存足够大，用手机下载也无所谓。之后可以拷贝到 Linux 电脑中编译。

**第 3 步，解压初始化包**

下载后的初始化包是一个.tar的压缩包，既可以使用命令行解压，也可以用操作系统的图形化工具解压。解压命令如下：

```
tar xf aosp-latest.tar
```

由于压缩包非常大，解压需要一定的时间，这一步建议使用图形化工具解压，方便我们查看进度。

解压完成后的初始化包，只保留了`.repo`目录，使用快捷键`Ctrl+H`显示隐藏文件。执行 repo sync 即可拉取Android主干分支的源码。

```bash
cd aosp
repo sync 
# 或 repo sync -l 仅checkout代码
```

此后，每次只需运行 `repo sync` 即可保持与主分支同步。

不过我们并不需要主干分支的源码，我们需要选择同步指定版本的Android源码，继续如下的操作

**第 4 步，同步指定分支的源码**

首先切换到`.repo/manifests`目录，再使用`git fetch`指令，拉取最新的远程代码库。

```bash
cd .repo/manifests
git fetch --all
git branch -r
```

使用git branch -r查看目前最新分支，这里我们选择android 13的分支，执行repo指令，将当前源码库从主干分支切换到Android 13的分支上。

```csharp
repo init -b android-13.0.0_r20
```

最后，执行repo sync 同步源码，就可以得到完整的Android 13源码了。

```bash
repo sync
```

------

#### 第 2 种方式，使用 repo 直接同步源码

**第 1 步，建立工作目录**

同样的，在开始下载AOSP源码之前，需要在合适的位置创建一个目录用于放置源码，使用如下命令：

```bash
mkdir AOSP
cd AOSP
```

**第 2 步，初始化仓库**

使用repo init 指令初始化源码仓库。

```bash
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest
```

**第 3 步，同步源码**

执行repo sync 指令同步源码。

```bash
repo sync
```

如果提示无法连接到 gerrit.googlesource.com，可以将以下内容配置到系统的环境变量中，然后重启操作系统。

```ini
ini
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```

如果需要同步指定的Android版本，只需要在repo init之后加上-b带上分支名称即可。

```bash
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-11.0.0.0_r40
repo sync
```

有关repo的更多使用方式，可以详细阅读中科大或清华大学的官方文档，这里不再赘述。

------

### Android 源码文件目录结构

- package

应用层（Application）源码。系统应用就在这里了，比如说系统设置，桌面，相机，电话之类的。

- frameworks

系统框架（Framework）层源码。

- hardware

硬件抽象（HAL）层源码。

- out

输出目录。编译以后生成的目录，相关的产出就在这里了

- build

用于构建Android系统的工具。也就是用于编译Android系统的

## 编译 AOSP

1. ### 配置编译环境

编译Android源码会用到很多第三方库，我们需要先将这些库配置好，指令如下

```css
sudo apt update
sudo apt install flex bison build-essential zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc fontconfig -y
sudo apt install make git-core gnupg zip unzip curl python3 openjdk-11-jdk -y
sudo apt install python-is-python3
sudo apt clean && sudo apt autoremove -y
```

1. ### 编译 Android 镜像

**第 1 步，加载环境变量**

在Android源码根目录执行`source build/envsetup.sh`，该指令会将`lunch`以及其他程序和变量添加到shell环境中。

```bash
cd aosp
source build/envsetup.sh
```

**第 2 步，选择编译的目标**

因为是在电脑上调试编译出的版本，所以这里我们选择 car_x86_64-userdebug。

> 执行lunch指令之前，当前窗口的shell环境必须已经执行过`source build/envsetup.sh`

lunch 打开选择菜单，选择car_x86_64-userdebug也就是 52。

```
lunch
67 # 选择67 编译车载系统
m #m会自动选择适合的线程进行编译 或者make -j4
emulator # 启动模拟器 使用编译好的系统
# 或者使用真机 进行刷机
```

也可以直接输入`lunch 52`/`lunch car_x86_64-userdebug`跳过列表选择，不同版本的Android源码项目，目标数字对应的目标并不一样，要注意选择。

**第 3 步，执行编译**

make 指令是aosp的编译指令，支持并发编译。可以使用-j指定并发编译的线程数。电脑的CPU核心数越多，可以设定的线程数就越大，编译速度也就越快，一般可以设为CPU核心数*4。如果不指定，构建系统会自动选择当前操作系统最适合的并行线程数。

```go
make
```

在调用 make 命令时，如果没有指定任何目标，则将使用默认的名称为“droid”目标，该目标会编译出完整的 Android 系统镜像。

**第 5 步，启动模拟器**

编译的时长取决于电脑的性能，首次编译约需要耗时约2-5个小时，控制台提示 build completed successfully则表示编译成功了。然后执行emulator就可以启动模拟器了。

```
emulator
```

等待开机动画播放完毕，看到Launcher界面，就表示编译成功了。如果编译后出现模拟器黑屏无法启动，可以再执行lunch sdk_car_x86_64-userdebug，然后再make一次。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/375694/1685233854689-3d6e3245-bdb3-4771-b816-7c5e140db31b.png#averageHue=%2306232c&clientId=u6246308f-eb7f-4&from=paste&height=429&id=bAWCl&originHeight=858&originWidth=1394&originalType=binary&ratio=2&rotation=0&showTitle=false&size=186883&status=done&style=none&taskId=u02521b53-4bce-48e3-8b88-529f47a0f64&title=&width=697)
> 注意这里用的是模拟器，选择编号**67**进行编译

- user: 限制所有权限，发布版
- userdebug: 允许root
- eng: 开发版，开放所有权限

![image.png](https://cdn.nlark.com/yuque/0/2023/png/375694/1685234036991-c0380fc8-9ce7-4c38-a339-e4e0448386ee.png#averageHue=%2307232c&clientId=u6246308f-eb7f-4&from=paste&height=419&id=KZvc3&originHeight=838&originWidth=1672&originalType=binary&ratio=2&rotation=0&showTitle=false&size=160146&status=done&style=none&taskId=u1d9ade59-e008-45b9-b8c4-2c447a6cd07&title=&width=836)
修改Android源码后如何进行编译？进入到指定的目录使用mm或者mmm指定修改的目录进行编译
```
mm # 编译当前模块
mmm # 编译指定模块 mmm packeage/apps/Luncher3
```
##### 编译android系统源代码特定模块
```c
m：  编译所有的模块
mm： 编译当前目录下的模块，当前目录下要有Android.mk文件
mmm：编译指定路径下的模块，指定路径下要有Android.mk文件

使用这些命令前需要在android源码根目录执行source build/envsetup.sh脚本设置环境
举例子，编译Camera2模块步骤：
1.源代码目录下执行source build/envsetup.sh脚本设置环境
2.直接执行mmm packages/apps/Camera2/
或者等价于
1.源代码目录下执行source build/envsetup.sh脚本设置环境
2.先切换目录cd packages/apps/Camera2/
3.执行mm
最后使用make snod重新生成sysem.img步骤：
1.source build/envsetup.sh
2.lunch 然后选择2   #选择2是选择和编译整个系统时的选择一致。
3.make snod           #先执行lunch，不然make snod报错build_image.py - ERROR。
```

##### 清理编译中间产物，以便重新开始编译
```c
1、在源码目录的根目录下，执行make clean;
2、进到源码的\linux\kernel\目录下，执行make mrproper；
3、再退回到根目录，执行source build/envsetup.sh, lunch, make -j16；
```

##### 生成工程信息文件 支持源码导入到AS中
依次执行以下命令，生成IDE工程信息文件 **android.ipr** 和 **android.iml**
```bash
source build/envsetup.sh
mmma development/tools/idegen
development/tools/idegen/idegen.sh
```
此时如果直接使用Android studio打开`android.ipr`，会特别慢，因为太多东西了，一直在索引。所以打开之前我们需要编辑一下`android.iml`配置文件。

我们可以使用下面的配置文件替代原始的配置文件，加快导入速度。该配置文件要求Android Studio只引入package模块的源码。如果有需要也可以引入frameworks模块的源码。

```
<?xml version="1.0" encoding="UTF-8"?>
<module version="4" relativePaths="true" type="JAVA_MODULE">
  <component name="FacetManager">
    <facet type="android" name="Android">
      <configuration />
    </facet>
  </component>
  <component name="ModuleRootManager" />
    <component name="NewModuleRootManager" inherit-compiler-output="true">
    <exclude-output />
    <content url="file://$MODULE_DIR$">
      <sourceFolder url="file://$MODULE_DIR$/packages" />
      <excludeFolder url="file://$MODULE_DIR$/frameworks" />
      <excludeFolder url="file://$MODULE_DIR$/.repo" />
      <excludeFolder url="file://$MODULE_DIR$/external/bluetooth" />
      <excludeFolder url="file://$MODULE_DIR$/external/chromium" />
      <excludeFolder url="file://$MODULE_DIR$/external/icu4c" />
      <excludeFolder url="file://$MODULE_DIR$/external/webkit" />
      <excludeFolder url="file://$MODULE_DIR$/frameworks/base/docs" />
      <excludeFolder url="file://$MODULE_DIR$/out/eclipse" />
      <excludeFolder url="file://$MODULE_DIR$/out/host" />
      <excludeFolder url="file://$MODULE_DIR$/out/target/common/docs" />
      <excludeFolder url="file://$MODULE_DIR$/out/target/common/obj/JAVA_LIBRARIES/android_stubs_current_intermediates" />
      <excludeFolder url="file://$MODULE_DIR$/out/target/product" />
      <excludeFolder url="file://$MODULE_DIR$/prebuilt" />
      <excludeFolder url="file://$MODULE_DIR$/external/emma" />
      <excludeFolder url="file://$MODULE_DIR$/external/jdiff" />
    </content>
    <orderEntry type="sourceFolder" forTests="false" />
    <orderEntry type="inheritedJdk" />
  </component>
</module>

```
#### android 系统映射文件说明
| **img名称** | **作用** |
| --- | --- |
| boot.img | 内核引导启动有关镜像 |
| ramdisk.img | 文件系统有关镜像，包含android系统文件根目录等 |
| system.img | android系统核心镜像，包含framework、系统app等，挂载在/system下 |
| userdata.img | 应用程序数据镜像，挂载在/data下 |
| cache.img | 临时数据存放镜像 |

### 编译过程遇到的问题
####  For the sprintf deprecation issue
[Error in compiling android (12) aosp on mac 13.3 m1 max](https://stackoverflow.com/questions/76129351/error-in-compiling-android-12-aosp-on-mac-13-3-m1-max)
Update: For the sprintf deprecation issue, you can actually fix it by replacing with snprintf accordingly, since on Xcode 14+, sprintf is deprecated on mac. There are a few repos which require this patching.
#### error: 'TARGET_OS_SIMULATOR' is not defined
[ICU-21513 check if TARGET_OS_SIMULATOR has been defined by mojca · Pull Request #1608 · unicode-org/icu](https://github.com/unicode-org/icu/pull/1608)
external/icu/icu4c/source/common/putil.cpp:1364:35: error: 'TARGET_OS_SIMULATOR' is not defined, evaluates to 0 [-Werror,-Wundef-prefix=TARGET_OS_]
#if U_PLATFORM_IS_DARWIN_BASED && TARGET_OS_SIMULATOR
#### failed to build some targets (05:22 (mm:ss)) ####
```cpp
#if U_PLATFORM_IS_DARWIN_BASED && defined(TARGET_OS_SIMULATOR) && TARGET_OS_SIMULATOR
# if !defined(ICU_DATA_DIR_PREFIX_ENV_VAR)
#  define ICU_DATA_DIR_PREFIX_ENV_VAR "IPHONE_SIMULATOR_ROOT"
# endif
#endif
```
#### too many file 的问题
[https://www.jianshu.com/p/d6f7d1557f20](https://www.jianshu.com/p/d6f7d1557f20)
[https://www.readfog.com/a/1663989524653510656](https://www.readfog.com/a/1663989524653510656)
macOS 解決 too many open files
報錯 too many open files 大致有以下三種可能 [[1]](https://bbs.huaweicloud.com/blogs/108323) [[2]](https://www.launchd.info/) [[3]](https://en.wikipedia.org/wiki/Launchd#launchctl)：

1. 操作系統打開的文件句柄數過多（內核的限制）
2. launchd 對進程進行了限制
3. shell 對進程進行了限制

內核的限制
整個操作系統可以打開的文件數受內核參數影響，可以通過以下命令查看
```
$ sysctl kern.maxfiles
$ sysctl kern.maxfilesperproc
```
在我電腦上輸出如下
```
kern.maxfiles: 49152
kern.maxfilesperproc: 42576
```
好像已經很大了。
如果需要臨時修改的話，運行如下命令
```
$ sudo sysctl -w kern.maxfiles=20480 # 或其他你選擇的數字
$ sudo sysctl -w kern.maxfilesperproc=18000 # 或其他你選擇的數字
```
永久修改，需要在 /etc/sysctl.conf 里加上類似的下述內容
```
kern.maxfiles=20480
kern.maxfilesperproc=18000
```
這個文件可能需要自行創建
launchd 對進程的限制
獲取當前的限制：
```
$ launchctl limit maxfiles
```
輸出類似這樣：
```
maxfiles    256            unlimited
```
其中前一個是軟限制，後一個是硬件限制。
臨時修改：
```
$ sudo launchctl limit maxfiles 65536 200000
```
系統範圍內修改則需要在文件夾 /Library/LaunchDaemons 下創建一個 plist 文件 limit.maxfiles.plist：
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>limit.maxfiles</string>
    <key>ProgramArguments</key>
    <array>
      <string>launchctl</string>
      <string>limit</string>
      <string>maxfiles</string>
      <string>65536</string>
      <string>200000</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>ServiceIPC</key>
    <false/>
  </dict>
</plist>
```
修改文件權限
```
$ sudo chown root:wheel /Library/LaunchDaemons/limit.maxfiles.plist
$ sudo chmod 644 /Library/LaunchDaemons/limit.maxfiles.plist
```
載入新設定
```
$ sudo launchctl load -w /Library/LaunchDaemons/limit.maxfiles.plist
```
shell 的限制
通過下述命令查看現有限制
```
$ ulimit -a
```
得到如下輸出
```
...
-n: file descriptors                256
...
```
通過 ulimit -S -n 4096 來修改。如果需要保持修改，可以將這一句命令加入你的 .bash_profile 或 .zshrc 等。
一般來說，修改了上述三個限制，重啓一下，這個問題就可以解決了。
在實際操作中，我修改到第二步重啓，第三個就自動修改了。

#### error: 'sprintf' is deprecated 的问题
看起来是 sprintf 方法已经废弃了，建议使用 snprintf 替代。嗯，🤔，作为新手还是先不改代码了，先查一下这个问题怎么绕过。
在项目仓库找到一个 [Issus #75](https://github.com/rockchip-linux/rkdeveloptool/issues/75)，移除 Makefile.am 里的 -Werror，然后重新执行：

```bash
autoreconf -i
./configure
make
```
或者使用SDK 10.14 或 10.15  11.3 下载地址：
[https://github.com/phracker/MacOSX-SDKs/releases/tag/11.3](https://github.com/phracker/MacOSX-SDKs/releases/tag/11.3)
解压缩放到：目录中：
```bash
cd /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/

```

将SDK的版本添加 11.3的SDK 版本 不要添加12以上的SDK 否则编译报错

1. 错误1 - LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection
```c
输入以下命令，移除代理，再执行命令
git config --global --unset http.proxy
```

2. 错误2 - 找不到高通芯片有关的符号连接
```c
internal error: could not open symlink hardware/qcom/sm8150p/Android.bp; its target (gps/os_pickup.bp) cannot be opened
internal error: could not open symlink hardware/qcom/sm7250/Android.bp; its target (gps/os_pickup.bp) cannot be opened
internal error: could not open symlink hardware/qcom/sm7150/Android.bp; its target (gps/os_pickup.bp) cannot be opened
internal error: could not open symlink hardware/qcom/sm8150/Android.bp; its target (data/ipacfg-mgr/os_pickup.bp) cannot be opened
```
解决方法：

3. 错误3 - 没有对应支持的sdk版本
```c
internal error: Could not find a supported mac sdk: ["10.10" "10.11" "10.12" "10.13" "10.14"]
```
解决方法：

4. 错误4 - 找不到高通芯片有关的文件或目录
```c
FAILED: 
hardware/qcom/sdm845/Android.mk:1: error: hardware/qcom/sm7150/Android.mk: No such file or directory
```
解决方法：

5. 错误5 - 出现这个原因可能和sdk版本有关
```c
system/core/base/cmsg.cpp:78:21: error: use of undeclared identifier 'PAGE_SIZE'
if (cmsg_space >= PAGE_SIZE) {
```
解决方法：
