# 编译AOSP源码

在Ubuntu里面新建一个enviroment.sh脚本，将以下内容复制进去

```
#base tools of ubuntu
sudo apt install net-tools gitk tree vim terminator synergy expect minicom cutecom 
#openjdk
sudo apt-get install openjdk-8-jdk 
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip libc6-dev tofrodos python3-markdown libssl-dev libxml-simple-perl  mingw-w64
#install adb and fastboot
sudo apt-get install android-tools-adb android-tools-fastboot
#install ssh server in order to enable ssh login and scp
sudo apt-get install openssh-server openssh-client
#install python
sudo apt-get install python python-dev python-protobuf protobuf-compiler python-virtualenv python-pip
#install libssl-dev to fix such error "fatal error: openssl/opensslv.h: No such file or directory"
sudo apt-get install libssl-dev
#install audit2allow
sudo apt-get install policycoreutils-python-utils
sudo apt-get install m4
#安装perl环境，否则可能会报错Can't locate XML/Simple.pm in @INC (you may need to install the XML::Simple module)
sudo cpan install XML::Simple
#解决no such file or directory以及Cannot generate supplementary makefiles的问题
sudo apt install lsb
sudo apt-get install lib32stdc++6
sudo apt-get install ia32-libs
sudo apt-get install lib32ncurses5
sudo apt-get install lib32z1

sudo chmod 777 enviroment.sh
```
运行脚本会自动下载安装所需环境，但因为Ubuntu默认的软件仓库地址在国内可能会方便比较慢，甚至会失败的情况，所以运行脚本前需要先设置一下软件仓库镜像源。
```
sudo gedit /etc/apt/sources.list 

```
打开镜像源文件，替换以下内容：
```
#添加阿里源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
#添加清华源
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse multiverse

```
替换完后，执行
```
sudo apt-get update
sudo apt-get -f install
sudo apt-get upgrade
```
更新完仓库地址之后再执行上面的脚本即可，安装过程中需要手动选择y确认安装，安装过程大概需要15分钟。

Android源码同时使用git和repo进行管理，repo是基于git的代码管理工具，类似github、gitee，所以需要同时安装git和repo
```
sudo apt-get update
sudo apt-get install repo
```
如果是ubuntu20.04，执行上述命令会提示无法定位repo包，那么这个时候需要手动安装repo
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
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

首先需要创建一个文件夹来存放源码
```
sudo mkdir /projects/Google-Android-Source/
sudo chmod 777 /projects/Google-Android-Source/
```
当然使用git还需要设置一下git全局配置
```
git config --global user.name "jason"
git config --global user.email "jason@163.com"
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest   #初始化仓库

#这里使用的是清华源，并且是master主干分支。
#google源是repo init -u https://android.googlesource.com/platform/manifest 需要FQ
#可以使用-b指定分支，比如：
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-10.0.0_r1
#查看所有分支地址：https://android.googlesource.com/platform/manifest/+refs
repo sync #这句话实际下载路径就是上面repo init时指定的路径。

#下载时间比较长，主要看网络给不给力了
#最后看到repo sync has finished successfully.表示下载完成。
#最新源代码android11大小约124G。。。
```
##### 查看并切换到指定分支
```c
cd .repo/manifests                   #来到这个目录
git branch -a       #查看所有分支
repo init -b android-12.0.0_r1  #回到工作目录，执行这句话切换到android-12.0.0_r1
repo sync               #切换完成再次执行同步操作
```

下载好源码开始编译：
```
source build/envsetup.sh #配置环境变量
lunch
67 # 选择67 编译车载系统
m #m会自动选择适合的线程进行编译 或者make -j4
emulator # 启动模拟器 使用编译好的系统
# 或者使用真机 进行刷机
```
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
```
3-1.sudo chmod 777 android.iml  #先给编辑权限
3-2.删除多余的orderEntry。全部excludeFolder或者多加一个frameworks的吧，都可以，后面可以改。
    配置大概如下：
<?xml version="1.0" encoding="UTF-8"?>
<module relativePaths="true" type="JAVA_MODULE" version="4">
<component name="FacetManager">
 <facet type="android" name="Android">
   <configuration />
 </facet>
</component>
<component name="NewModuleRootManager" inherit-compiler-output="true">
 <exclude-output />
 <content url="file://$MODULE_DIR$">
   <sourceFolder url="file://$MODULE_DIR$/frameworks/base" isTestSource="false" />
   <excludeFolder url="file://$MODULE_DIR$/.repo" />
   <excludeFolder url="file://$MODULE_DIR$/BUILD" />
   <excludeFolder url="file://$MODULE_DIR$/art" />
   <excludeFolder url="file://$MODULE_DIR$/bionic" />
   <excludeFolder url="file://$MODULE_DIR$/bootable" />
   <excludeFolder url="file://$MODULE_DIR$/compatibility" />
   <excludeFolder url="file://$MODULE_DIR$/cts" />
   <excludeFolder url="file://$MODULE_DIR$/dalvik" />
   <excludeFolder url="file://$MODULE_DIR$/developers" />
   <excludeFolder url="file://$MODULE_DIR$/development" />
   <excludeFolder url="file://$MODULE_DIR$/device" />
   <excludeFolder url="file://$MODULE_DIR$/external" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/av" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/base/apct-tests" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/base/docs" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/compile" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/ex" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/hardware" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/layoutlib" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/libs" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/minikin" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/multidex" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/native" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/opt" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/proto_logging" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/rs" />
   <excludeFolder url="file://$MODULE_DIR$/frameworks/wilhelm" />
   <excludeFolder url="file://$MODULE_DIR$/hardware" />
   <excludeFolder url="file://$MODULE_DIR$/kernel" />
   <excludeFolder url="file://$MODULE_DIR$/libcore" />
   <excludeFolder url="file://$MODULE_DIR$/libnativehelper" />
   <excludeFolder url="file://$MODULE_DIR$/out" />
   <excludeFolder url="file://$MODULE_DIR$/packages" />
   <excludeFolder url="file://$MODULE_DIR$/pdk" />
   <excludeFolder url="file://$MODULE_DIR$/platform_testing" />
   <excludeFolder url="file://$MODULE_DIR$/prebuilt" />
   <excludeFolder url="file://$MODULE_DIR$/prebuilts" />
   <excludeFolder url="file://$MODULE_DIR$/sdk" />
   <excludeFolder url="file://$MODULE_DIR$/system" />
   <excludeFolder url="file://$MODULE_DIR$/test" />
   <excludeFolder url="file://$MODULE_DIR$/toolchain" />
   <excludeFolder url="file://$MODULE_DIR$/tools" />
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
