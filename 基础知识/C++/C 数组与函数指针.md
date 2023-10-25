# C | 数组与函数zhi'zhen

### 数组常见的应用

在C语言中数组和java还是有很大的不同的，C语言数组越界不会报错可以无限扩展数组，数组没有边界，但是一般不会推荐这么做，我们来一探究竟

#### 一维数组

- C语言数组的特性

如下代码：虽然设置了arr数组的大小为6，但是设置7个或者多个，而且即便遍历数组超出了数组的长度，也不会报错。

```c
void forArray(int arr[], int size) {
    for (int i = 0; i < size; ++i) {
        //即便size大于数组的长度，也不会发生数组越界的问题
        printf("forArray:%d\n", arr[i]);
    }
}

int main() {
    int size = 10;
    int arr[6] = {1, 2, 3, 4, 5, 6,7};
    forArray(arr, size);
    return 0;
}
```

输出结果如下：数组大小为6遍历到7的时候并没有输出7，而是68不知道哪里来的一个值。(在开发中千万不要这样写，这里主要是验证C语言的数组和Java数组的区别)，C语言中数组不像Java一般严谨。

```c
forArray:1
forArray:2
forArray:3
forArray:4
forArray:5
forArray:6
forArray:68
forArray:0
forArray:10
forArray:0
```

- 遍历数组,C 语言数组遍历有多种方法如下

```c
    //下标法遍历指针数组
    for (int i = 0; i < 3; ++i) {
        printf("arr:%d\n", arr[i]);
    }
    //地址法遍历数组
    for (int i = 0; i < 3; ++i) {
        printf("arr:%d\n", *(arr + i));
    }
    //指针法
    int *c = arr;
    for (int i = 0; i < 3; ++i) {
        printf("arr:%d\n",*(c+i));
    }
```

- `arr `和 `&arr` 是否相等

这里非常重要在数组变量arr和&arr看起来是相等的，但是存在本质的不同。

arr和&arr他们的地址是一样的，相当于**两个指针指向同步一个地址，但是两个指针不一定是一致的**。

```c
    //arr == &arr
    printf("arr:%p\n", arr);//000000000061FDF0
    printf("&arr:%p\n", &arr);//000000000061FDF0
```

我们来验证一下, `*arr` 对`arr`解引用，根据字节大小我可以看到指向的类型是`int`，而对`&arr`解引用，得到的字节大小是24说明指向的类型是`int[6]`数组。

```c
 //两个指针指向同步一个地址，指向类型不一定一样
    printf("*arr=%d\n", sizeof(*arr));//*arr=4       指向类型 int  很显然arr只是数组的首地址 第一个元素 int = 1
    printf("*&arr=%d\n", sizeof(*&arr));//*&arr=24   指向类型 int[6] &arr 整个数组   6 * 4 = 24
```

当用指针指向数组时，一定要注意指针指向arr 和 &arr 是有本质的不同的。



- 指针数组，数组元素为指针类型, 这个很好理解，同理a  和 &a 还是存在本质的不同的。

```c
    int a1 = 10;
    int a2 = 20;
    int a3 = 30;
    int *a[3] = {&a1, &a2, &a3};
    /**
     *  a=000000000061FDC0
        &a=000000000061FDC0
     */
    printf("a=%p\n", a);
    printf("&a=%p\n", &a);
    printf("*a=%d\n", sizeof(*a));//8
    printf("*&a=%d\n", sizeof(*&a));//8*3 = 24
```



#### 二维数组

二维数组和一维数组存在着一些不同，我们来继续探究

在二维数组中`a` 和 `&a` 和 `*a` 他们三个地址是一样，但是他们本质上不同的和一维数组也不一样。

a: 表示的是第一行数组的首地址，其实它代表着二维数组中的一行；



&a : 表示整个二维数组的首地址，代表着整个二维数组；



*a : 表示第一行第一个元素的首地址，代表着第一个元素;



![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631760810350-ba5a1ae1-bbe2-4736-a938-1827d7d43ca4.png)

```c
int a[3][4] = {
            1, 2, 3, 4,
            5, 6, 7, 8,
            9, 10, 11, 12
    };

    //a 第一行数组的首地址   &a 整个二维数组首地址   *a 第一行第一个元素的首地址
    printf("a:%p\n", a);//000000000061FDE0
    printf("&a:%p\n", &a);//000000000061FDE0
    printf("*a:%p\n", *a);//000000000061FDE0
```

下面可以进行验证，分别打印a  &a  *a 指向类型的大小。**a -> \*a -> 16 (一行数组)**、**&a -> \*&a -> 48(整个二维数组)**、***a -> \**a -> 4 (第一个元素)**

```c
    printf("-----------------\n");
    //查看类型大小size
    printf("a-size:%d\n", sizeof(*a));//16  4*4 表示一行数组
    printf("a:%p\n", a);//000000000061FDE0
    printf("a+1:%p\n",a+1);//000000000061FDF0    +16 进1
    printf("-----------------\n");
    printf("&a-size:%d\n", sizeof(*&a));//48 12*4 表示整个数组
    printf("&a:%p\n", &a);//000000000061FDE0
    printf("&a+1:%p\n", &a+1);//000000000061FE10   +48
    printf("-----------------\n");
    printf("*a-size:%d\n", sizeof(**a));//4  表示数组中的一个元素
    printf("*a:%p\n", *a);//000000000061FDE0
    printf("*a+1:%p\n", *a+1);//000000000061FDE4   +4
    printf("-----------------\n");
```

关于二维数组中的a、&a、*a 它们之间的区别必须要搞懂，因为二维数组在开发中经常用到，尤其是音视频和图像处理等开发，在复习一下：

a: 代表着第一行数组

&a: 代表着整个二维数组

*a : 代码着第一行数组，第一个元素



- 二维数组取值的方式

二维数组取值通过数组下标取值或者通过指针进行取值，只要理解了a 、&a、*a三者的不同和本质很容易搞懂。

```c
    //数组取值法
    printf("value:%d\n",a[1][0]);//5
    //指针取值法
    printf("*a:%d\n", **a);//1  第一行第一个元素
    printf("*a:%d\n", **(a + 1));//5 第二行第一个元素
    printf("*a:%d\n", *(*(a + 1) + 1));//6 第二行第二个元素
```

- 数组指针与二维数组

如何将二维数组，赋值给数组指针，由数组指针操作二维数组呢？数组指针就是一个指针指向了**一行**数组(注意是一行)。

如下代码：在二维数组中表示一行的就是**a** ,主要搞懂了上述的本质区别，非常容易理解了。

```c
    int (*p)[4] = a;
    printf("p:%p\n", *p);//000000000061FDE0
    printf("p:%d\n", **p);//1
    printf("p:%p\n", *(p + 1));//000000000061FDF0
    printf("p:%d\n", **(p + 1));//5
    printf("p:%d\n", *(*(p + 1) + 1));//6
```

如果看着还懵逼的话，看下图所示：内存模型示意图

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631761835633-a6cb01c1-66b7-43af-85ce-2d6027e697fe.png)



### 函数指针

在C语言中函数也存在地址，但是函数指向的类型没有任何意义，函数指向的是一块代码区域。而对于C/C++一切皆指针，只要有地址存在一切都可以用指针来表示。



1. 函数的地址：fun  &fun *fun 三者的区别

它们三者的地址都是一样的，但是函数的地址指向没有任何意义，他就是表示了一块执行的代码区域。

```c
void func(int a,int b){
    printf("a:%d,b:%d\n",a,b);
}

int main(){
    func(1,2);
    //1. 函数存在地址的 函数的指向类型没有意义的，因为函数指向的一块代码区域
    printf("func = %p\n",func);//0000000000401550
    printf("&func = %p\n",&func);//0000000000401550
    printf("*func = %p\n",*func);//0000000000401550
    return 0;
}
```

1. 函数指针的定义

定义函数指针：返回值 (*指针名) (参数列表...)

```c
    void (*p) (int,int);
    p = func;//给指针赋值
    p(3,4);//a:3,b:4
```

1. 函数指针作为参数

函数指针可以作为参数类似Java中的回调

```c
void test(int a,int b,int (*callback) (int)){
    int result = a+b;
    callback(result);
}

int call(int result){
    printf("result:%d\n",result);
    return result;
}

//调用
test(30,40,call);//result:70
```

1. 定义函数别名

C语言中可以通过typedef定义函数别名

```c
//给函数定义别名
typedef void FUNC(int,int);
    //4. 定义函数别名简化函数指针
    FUNC *p2 = func;
    p2(1,2);//a:1,b:2
    (*p2)(1,2);//p2 == *p2
    (&func)(1,2);////a:1,b:2
    (*func)(1,2);////a:1,b:2
```

1. 变更函数指针的指向

记住：如果要通过一个函数来变更外部，一定要传递外部变量的地址,如果是指针那么函数要用二级指针作为函数接收。

如下代码，直接改变函数指针的指向

```c
int jia(int a, int b) {
    return a + b;
}

int jian(int a, int b) {
    return a - b;
}

typedef int PN (int,int);

int main(){
    PN *p = jia;
    printf("p:%d\n",p(1,2));//3
    p = jian;//修改函数指针的指向
    printf("p:%d\n",p(1,2));//-1
 	return 0;   
}
```

如下代码：通过函数改变外部的指向，下面的代码是**错误**的示例。

```c
void changeError(PN *p){
    p = jia;
}

//变成p为jia函数
changeError(p);//p->jian     jian -> jia 错误的，并不会改变p的指向

printf("p:%d\n",p(1,2));//-1  p还是减法没有变成加法
```

代码看着懵逼来看下图：内存模式示意图,只是改变了内部的指向，而对于外部的指向并没有变化。

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631782024844-9e83a0a4-d802-48f7-9270-99226f25d12f.png)



正确的做法，函数通过二级指针接收p的地址(注意是p的地址 &p)

```c
void change(PN **p){
    *p = jia;
}

    //如果要通过函数改变外部的变量，一定要传递外部变量的地址,如果是指针那么函数要用二级指针作为函数接收
    change(&p);//要改变p的指向，必须要传递&p

    printf("p:%d\n",p(1,2));//3  p还是减法没有变成加法
```

如下内存模型示意图：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631782312698-8cbb1c0c-816e-4772-8a0a-ae78b2a73189.png)



### 函数指针数组

函数指针数组和指针数组有什么区别呢？指针数组就是数组中存储的是指针类型，而函数指针数组则是数组中存储的函数指针类型

[4] -> [int (*) (int,int),int (*) (int,int),int (*) (int,int),int (*) (int,int)]

函数指针数组：适合应用在观察者集合，回调给各个模块

```c
int jia(int a, int b) {
    return a + b;
}

int jian(int a, int b) {
    return a - b;
}

int cheng(int a, int b) {
    return a * b;
}

int chu(int a, int b) {
    return a / b;
}

//函数指针数组
int main() {
    int *p[10];//指针数组

    ////函数指针数组   [4] -> [int (*) (int,int),int (*) (int,int),int (*) (int,int),int (*) (int,int)]
    int (*pn[4])(int, int) = {jia, jian, cheng, chu};
    int a = 10;
    int b = 20;
    for (int i = 0; i < 4; ++i) {
        /**
         * result:30 result:-10 result:200 result:0
         */
        printf("result:%d\n", pn[i](a, b));
    }

    return 0;
}
```

本篇讲解了数组和函数指针的重要性，在开发中会经常用到，函数指针被用作回调函数使用。