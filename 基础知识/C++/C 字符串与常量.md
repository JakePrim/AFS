# C | 字符串与常量

在C语言中，字符串和常量与其他语言有什么区别呢？C语言指针就是一切，那么指针和字符串与常量存在着什么关系呢？

### 字符串

在C语言中不存在显示的**字符串类型**，C语言中的字符串都是以**字符串常量**或存储在**数组**中，同时C语言提供了一系列库函数来操作字符串这些函数在string.h头文件中。

在C语言中字符串遇到`\0`才会结束。

1. 通过数组存储的字符串，存储在栈中

```c
    //数组char 在栈区
    char data[] = {'a', 'b', 'c', 'd', 'e', '\0'};//在数组中 需要手动加上\0
    printf("%s\n", data);//打印字符串 abcde
```

1. 通过数组在大括号中末尾不加上 \0 打印字符串会有问题，如下代码：

```c
    //如果不定义\0
    char data2[] = {'a', 'b', 'c', 'd', 'e'};//data2:abcdeabcde 直到遇到内存中的\0才会停止
    printf("data2:%s\n", data2);
    char data3[5] = {'h', 'e', 'l', 'l', 'o'};
    printf("data3=%s\n", data3);//data3=helloabcdeabcde
```

1. 通过数组直接等于字符串,默认会自动加一个\0,常见的写法

```c
    char data[] = "abc";//默认加了一个 \0
```

1. 通过指针指向的字符串，字符串存储在常量区

注意：常量区的内容不能修改，如果`*str="abc"` 会报错误. 通过`char *` 指针声明的字符串不用加`\0`,编译器会自动加上`\0`结束符号



%s 直接输入str即可，不用加上*

```c
    //指针定义的char abcde在常量区，注意：常量区的内容是不能修改的
    char *str = "abcde";//C 语言中在字符串默认加上 \0 表示字符串结束
    //%s 直接输入str即可，不用加上*
    printf("%s\n", str);//打印字符串 abcde
    while (*str != '\0') {
        //%c 需要加上*
        printf("%c ", *str++);
    }
    printf("\n");

    //凡是常量区定义的数据是不能修改的
//    *str = "hello"; //报错
```

数组声明的字符串和指针声明的字符串的区别如下图所示：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631945289378-6a69fbab-457a-454b-a1ce-5bb5452936f8.png)

1. scanf 接收控制台输入的字符串

注意：`scanf `遇到空格会换行输出，遇到回车结束输入，`scanf`会自动添加`\0`结束符号。

```c
 	//定义数组 通过scanf进行输入字符串看结果如何
    char data4[10];
    printf("input:\n");
    //控制台输入字符串
    scanf("%s", data4); //注意scanf 遇到空格就会输出一行，遇到回车结束
    printf("data4=%s\n", data4);//data4=hello   scanf会自动添加\0
```

1. gets 和scanf类似

`gets `和 `scanf`功能一样，不过gets会把空格当成字符串输出。

```c
char data4[10];    
gets(data4);//gets 会把空格当成字符串  遇到回车结束
printf("gets:%s\n",data4);
```

1. fgets

fgets 也是接收控制台输入的字符串，在上述gets和scanf函数都需要固定字符串数组的大小，如果不确定字符串数组的大小要怎么做呢？fgets 就是以固定的长度为一组，循环输出

```c
    //以固定的长度为一组 而不是根据数组大小
    while (1) {
        char str[10];
        fgets(str, 10, stdin);
        printf("str=%s\n", str);
    }
```

输入和输出：以10个字符串数组长度进行分组输出

```c
input:
hello world this is fgets out 10

输出方式如下：
str=hello wor
str=ld this i
str=s fgets o
str=ut 10
```

1. `gets `或者 `scanf `能接受一个`char *` 的指针吗？

在上述提到过，char * 声明的字符串是在常量区，而常量区的内容是不可以修改的，所以如果gets、scanf接受一个char * 运行就会报错。但是**指针的指向可以改变，指针指向的内容不能通过指针来进行修改**。

```c
    //注意 输入不能使用char *   因为char *声明的字符串是在常量区，而且常量区是不能被修改的
    char *s1 = "abc";
    *s1 = "a";//这样是不行的，他是向改变常量的内容
    s1 = "bcd";//这样可以的，改变了指针的指向，并没有改变常量
    gets(s1);//发生错误，gets尝试的就是*s1 = xxxx 要改变常量的内容 所以报错了
    printf("s1:%s\n",s1);
```



### 常量指针和指针常量

在C语言中常量的声明通过关键字const，除了普通的常量也有常量指针和指针常量，在上述声明字符串char *str = "abc"; str就是一个常量指针，因为"abc" 存储在常量区，指针指向了一个常量。

\* 和 const 谁在前面，谁先读：* 指针、const 常量



常量指针：指向常量的指针，不能指向变量，它指向的内容不能被改变，不能通过指针来修改它指向的内容，但是指针自身不是常量，它自身的值可以改变，指向另一个常量。

```c
    //常量指针:指向了常量，可以改变指向，但是不能改变地址的内容
    char *s = "abc";
    const int *ptr;//const 离 *ptr最近 不能修改
    int const *ptr2;//const 离 *ptr2最近 不能修改
```

指针常量：指针本身是常量，它指向的地址不可改变，但是地址的内容可以通过指针来改变，那么指针常量在定义时必须赋值，并且指向的地址伴随它一生.

```c
    //指针常量:指针是一个常量，指向的地址不能修改，指向地址的内容可以修改
    int a = 2;
    int * const b = &a;//b就是指针常量，const 离谁最近 谁就不能修改  b指向的地址不能修改
```

### 字符串的常用操作

1. 字符串的长度

字符串的长度有两种方式：

1. sizeof 获取字符串的长度，但是这种方式获得不准确为什么呢？因为在C语言中字符串默认会有一个`\0`结束符，而`sizeof`会将`\0`也算进去，算出的长度会大于1。
2. strlen() 函数：可以获取实际的字符串长度，获取\0结束符之前的字符串的大小，但是如果遇到 `"abc\0de"` `strlen`获取到的大小就是3，遇到`\0`结束。

```c
    char *s = "abc";
    printf("string size:%d\n", strlen(s));//string size:3
    printf("s sizeof:%d\n", sizeof(s));//8
    printf("data sizeof:%d\n", sizeof(data));//4 多出来的一个就是 \0
```

1. strcpy 字符串拷贝函数

`strcpy `将目标字符串进行替换，例如：hellotest -> jake 将jake复制到hellotest 那么结果就是：jake\0test.

注意：目标字符串的大小不会发生改变，`strlen `遇到`\0`结束，所以strlen不一定等于目标字符串的大小

如下图所示：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632360343271-91c7973b-4501-4415-82f3-987d95c3d676.png)

字符串注意最好是数组的形式，不能是常量指针，因为常量区的内容不能改变

```c
    char s1[] = "hellotest";//存储在栈区
    printf("len:%d\n", strlen(s1));//9  strlen 同理遇到\0就计算结束
    printf("sizeof:%d\n", sizeof(s1));//10 数组的大小多了一个\0    
//将jake复制到s1
    strcpy(s1,"jake");//此处操作实际是：jake\0test  前面的hello被替换掉了

    printf("%s\n",s1);//jake  输出字符串 遇到了\0就会自动结束
    printf("len:%d\n", strlen(s1));//4  strlen 同理遇到\0就计算结束
    printf("%d\n",s1[4]);//第五个元素为0  在C语言中字符串遇到\0就会结束
    printf("%c\n",s1[5]);//第六个元素 t
    printf("sizeof:%d\n", sizeof(s1));//10 数组的大小没有改变
```

如下所示：使用常量指针会运行报错

在声明字符串时一定要注意是数组的形式声明的字符串存储在栈区内容可以改变，如果使用的是指针的形式声明的字符串存储在常量区，常量区的内容不可以改变

```c
  char *s3 = "abc";
  strcpy(s3,"d"); //出现错误 strcpy无法传递
  printf("s3:%s\n",s3);
```

1. 字符串的拼接

C 语言中如何实现字符串的拼接呢？在java语言中可以直接使用+号进行拼接字符串，而在C语言中是不行的

```c
printf("%s\n","a"+"B");//错误
```

可以通过指针的形式进行拼接字符串，将目标指针移动至尾部然后++拼接

注意还是要注意指针形式声明的字符串，内容是不可以改变的，注意 注意 注意！！！ 重要的事情说三遍

```c
//字符串拼接
void mystrcat(char *s1,char *s2){
    //s1和s2指向的是存储在栈区的字符串 所以可以通过指针进行操作
    while (*s1) s1++;//while(\0) 循环结束   s1 -> \0
    //修改指针 指向的内容   s1 -> \0 -> *s2
    while (*s1++=*s2++);//s1 = \0 结束循环
}
    char s4[] = "abc";//栈
    char s5[] = "123";//栈

    //字符串拼接
    //如果是 char * s4="abc"  char * s5 = "123" 这样是不行的 因为常量区的内容不能进行修改
    mystrcat(s4,s5);
    printf("s4:%s\n",s4);//s4:abc123
    printf("s4:%d\n",s4[6]);//0
```

1. 字符串数组

字符串本身就是一个数组，那么字符串数组就是一个二维数组。

字符串数组：指针常量

```c
    //指针常量
    char arr[4][6] = {"abc", "def", "123", "wer"};
    //指针常量  arr -> * const
//    arr[0] = "sds";//不行的 不能修改指向  arr[0] = "abc" 是一个指针常量
    printf("arr[0]:%s\n", arr[0]);//arr[0]:abc
    //可以修改指向的内容
    strcpy(arr[0], "aabbcc");//arr[0]:aabbcc  可以修改内容  不可以修改指向的地址：指针常量
    printf("arr[0]:%s\n", arr[0]);
```

字符串数组：常量指针

```c
 //常量指针  指针数组 arr1[0]:常量指针 { char *,char *,char *,char * }
    char *arr1[4] = {"abc", "123", "sd", "se"};
    arr1[0] = "abd";//可以修改指向，不可以修改指向的内容  定价于 char * arr1 = "abc" 正是一个常量指针
    printf("value:%s\n", arr1[0]);//value:abd
    strcpy(arr1[0], "aabbcc");//修改内容 报错
    printf("value:%s\n", arr1[0]);
```

在强调一遍：

指针常量：修饰的是指针，指针指向的地址不能改变，而指针指向的内容可以改变。

常量指针：修饰的是内容，指向指向的地址可以改变，而指针指向的内容在常量区不能通过指针的方式来改变。

### 结构体

结构体和javaBean 类似，一般用于定义modle

```c
//结构体 类似java的javabean
    struct stu{
        char *name;
        int num;
        int age;
        char group;
        float scope;
    } person3;//声明时设置结构体

    //类似匿名 内部类
    struct{
        char *name;//8

        int num;//4
        int age;//4

        //这两个也是按照8个字节来算的
        char group;//1
        float scope;//4
    } person4;//声明时设置结构体

    //定义结构体
    struct stu person,person2;
```

计算结构体的占用字节大小

结构体如何计算占用的字节大小呢？如下代码在64位机器上，指针占8个字节，int 占4个字节，char占1个字节，float占四个字节，double占8个字节。

```c
    struct stu{
        char *name;//8
        int num;//4
        int age;//4
        char group;//1
        float scope;//4
    } person;
printf("person:%d\n", sizeof(person));//总大小：24字节
```

但是上述代码的运行结果是24个字节，实际上我们计算的是8+4+4+1+4 = 21.C语言一般读取内存在64位的机器上是一次取8个字节进行读取，如果超出则移动到下一个8个字节中。

如下图所示：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632384303555-c7224fcd-923b-46e8-8d86-90cab08e07a6.png)

结构体成员的对齐规则：

1. 结构struct或联合union的数据成员，第一个数据成员放在offset为0的地方。
2. 每个数据成员存储的起始位置要从该成员本身大小的整数倍开始(比如int在64位机上为4个字节，则要从4的整数倍地址开始存储,0是任何数的整数倍)

如下面的代码所示：char 为1个字节，int为4个字节，a存放在offset 为0的位置，b的起始位置为1，但是不满足int为4个字节的整数倍，因此要在a后面补齐，是b的其实位置为4。

```c
    struct node{
        char a;
        int b;
    }node;
    printf("node:%d\n", sizeof(node));//8
    struct pa{
        char a;//1
        struct node b;//8  int 4  char 1
    };
    struct pa p1;
    printf("p1:%d\n", sizeof(p1));//12

    struct node2{
        double a;//8
        char c;//1
    };
    struct node2 n2;
    //最大类型的整数倍
    printf("n2:%d\n", sizeof(n2));//16
```

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632384150367-cd0df3b1-919d-4e65-8b2b-69f25e7957ff.png)