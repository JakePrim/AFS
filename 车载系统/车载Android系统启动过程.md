# 车载Android系统启动过程

> 注意在学习车载专题之前，你需要清楚Farmework源码及流程。

从启动原理来看Android Automotive和Android有什么区别？
![](./%E8%BD%A6%E8%BD%BDAndroid%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B.assets/1683293549715-6b6f425d-5496-43f2-a9e0-95bfbe326e0d.jpeg)
[Framework | init进程原理](https://www.yuque.com/jakeprim/android/gmgsed?view=doc_embed)

```xml
-> % adb shell
munch:/ $ ps
USER           PID  PPID     VSZ    RSS WCHAN            ADDR S NAME
shell        10969  9181 2178780   2996 __arm64_s+          0 S sh
shell        10971 10969 2185620   3832 0                   0 R ps
munch:/ $ ps -el/ef
```

查看当前设备的进程:例如常见的zygote进程和system server进程。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/375694/1683508232195-6008ca0a-93b4-4be5-8eec-54fc28593faf.png#averageHue=%23191919&clientId=u9d7145b5-07c9-4&from=paste&height=62&id=u46ea9690&originHeight=124&originWidth=656&originalType=binary&ratio=2&rotation=0&showTitle=false&size=18606&status=done&style=stroke&taskId=ub6a31beb-6a10-4498-b7c5-bc3809e1fd8&title=&width=328)