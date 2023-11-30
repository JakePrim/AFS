# JNI 开发指南

> JNI :Java Native Interface Java本地接口，用于Java和其他类型语言(C、C++等)交互，JNI 是Java调用Native的语言特性，属于Java的和Android无直接关系，例如驱动程序都是C/C++开发的，通过JNI调用C/C++的实现驱动的代码，扩展Java虚拟机的能力，另外在高效率的数学运算、游戏的实时渲染、音视频的编码和解码一般都是用C开发的。

JNI和NDK关系：

- JNI 是Java平台JDK提供的一套非常强大的框架
- NDK Android平台提供的Native开发工具集开发包Native Development Kit(gcc/g++等)
- NDK对JDK的JNI进行了二次封装

Java的数据类型通过JNI转换为C/C++类型，包括JNI返回给Java的类型转换等。

![JNI](./JNI%20%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97.assets/Snipaste_2023-11-06_15-10-58.png)

C/C++调用Java类对象的属性和方法的签名规则：

| boolean  | Z                   |
| -------- | ------------------- |
| byte     | B                   |
| char     | C                   |
| short    | S                   |
| int      | I                   |
| Float    | F                   |
| double   | D                   |
| void     | V                   |
| object   | L完整的类名;        |
| 数组类型 | int[] [I double [[D |



Activity的代码如下：

```java
public class MainActivity extends AppCompatActivity {

    // Used to load the 'ndkdemo' library on application startup.
    static {
        System.loadLibrary("ndkdemo");
    }

    private ActivityMainBinding binding;

    private int num = 1;

    private TextView tv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
        tv = binding.tv;
        tv.setOnClickListener(view -> stringFromJNI());
//        tv.setText(stringFromJNI());
    }

    public native void stringFromJNI();
}
```



JNI 的代码如下：

```cpp
[[include]] <jni.h>
[[include]] <string>
[[include]] <android/log.h>

[[define]] TAG "sufulu" //这是自定义log的标识
[[define]] LOGD(...)  __android_log_print(ANDROID_LOG_DEBUG,TAG,__VA_ARGS__) //定义LOGD类型

/**
 * 参数一：JNIEnv* env 表示指向可用JNI函数表的接口指针，所有跟jni相关的操作都需要通过env来完成
 * 参数二：jobject 是调用该方法的Java对象，这里是MainActivity调用的，所有thiz代表MainActivity
 */
void native_jniTest(JNIEnv *env,jobject thiz) {
    //获取MainActivity的class对象
    jclass clazz = env->GetObjectClass(thiz);

    //参数1：MainActivity的class对象
    //参数2: 变量名称
    //参数3: 变量类型-方法签名
    //获取num变量的属性ID
    jfieldID numFieldId = env->GetFieldID(clazz, "num", "I");
    //获取num变量的值
    jint num = env->GetIntField(thiz, numFieldId);
    //改变num的值
    env->SetIntField(thiz, numFieldId, num + 1);


    //获取tv的变量id
    jfieldID tvFieldId = env->GetFieldID(clazz, "tv", "Landroid/widget/TextView;");
    //获取tv的实例对象
    jobject tvObject = env->GetObjectField(thiz, tvFieldId);
    //获取textView的class对象
    jclass tvClass = env->GetObjectClass(tvObject);
    //获取setText方法ID
    jmethodID setTextMethodId = env->GetMethodID(tvClass, "setText", "([CII)V");
    //获取setText所需的参数
    //将num转化为String
    char buf[64];
    sprintf(buf, "%d", num);
    jstring pJstring = env->NewStringUTF(buf);
    const char *value = env->GetStringUTFChars(pJstring, JNI_FALSE);
    //创建char数组，长度为字符串num的长度
    jcharArray charArray = env->NewCharArray(strlen(value));
    //开辟jchar内存空间
    jchar *pArray = (jchar *) calloc(strlen(value), sizeof(jchar));
    //将num字符串缓冲到内存空间中
    for (int i = 0; i < strlen(value); ++i) {
        *(pArray + i) = *(value + i);
    }
    //将缓冲的值写入到上面的char数组中
    env->SetCharArrayRegion(charArray, 0, strlen(value), pArray);
    //调用setText方法
    env->CallVoidMethod(tvObject, setTextMethodId, charArray, 0, env->GetArrayLength(charArray));
    //释放资源
    env->ReleaseCharArrayElements(charArray, env->GetCharArrayElements(charArray, JNI_FALSE), 0);
    free(pArray);


//    std::string hello = "Hello from C++";
//    return env->NewStringUTF(hello.c_str());
}

static const JNINativeMethod nativeMethod[] = {
        {"stringFromJNI", "()V", (void *) native_jniTest},
};

/**
* 动态注册
*/
extern "C"
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reversed) {
    JNIEnv *env = NULL;
    //初始化JNI
    if (vm->GetEnv((void **) &env, JNI_VERSION_1_4) != JNI_OK) {
        return JNI_FALSE;
    }
    //找到需要动态注册的java类
    jclass jniClass = env->FindClass("com/example/ndkdemo/MainActivity");
    if (nullptr == jniClass) {
        return JNI_FALSE;
    }
    //动态注册
    if (env->RegisterNatives(jniClass,nativeMethod,sizeof(nativeMethod)/sizeof(nativeMethod[0])) != JNI_OK){
        return JNI_FALSE;
    }
    //返回jni版本
    return JNI_VERSION_1_4;
}
```

