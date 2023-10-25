# C | 复杂函数与IO操作

### 复杂函数如何解读?

在C/C++开发中，经常会遇到非常复杂的函数声明比如：`int *(*(*pfn)(int *))[10];` ，这个到底是什么函数？看着一脸懵逼，，，

这边文章就带领大家，轻松读懂这种复杂函数。

在C语言中复杂函数有一个注明的解析法则：

右左解析法则：首先从变量名圆括号起，pfn先向右看，遇到圆括号，则调转方向，向左看*，这就说明pfn是一个指针。当括号内的内容解析完毕，就跳出括号，一直重复这个过程直到表达式解析完毕，其实复杂的函数离不开指针函数、函数指针、指针数组、数组指针。



解读复杂函数并不难，先从最简单的开始：

根据右左法则，直接解读出来的是，func 是一个指针，指向了一个函数(int),函数的返回值是int类型

```c
int (*func)(int);
```

如下推导图所示：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632469906048-ab94c553-0405-4184-9f0a-165e7c470499.png)



再来看一个稍微复杂点的函数：

这个函数并不难，可以推导出：fun2 是一个数组，数组的元素存放指针,元素指向的是一个函数(int *)，函数的返回值int

```c
int (*fun2[5])(int *p);
```

如下示意图所示：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632470830604-cf2ad6fa-cdbd-4c08-9174-9d05ef402c6f.png)



再来看一个如下代码：

根据右左法则：首先可以看到funn3是一个指针，指向了数组有5个元素[5] 所以fun3就是一个数组指针，数组中的元素类型是一个指针，指向了一个函数(int *p) 函数的返回值是int

```c
int (*(*func3)[5])(int *p);
```

直接看下面的示意图就一目了然了：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632470849381-a59564cc-269a-4d8e-95fb-76b5e8a13934.png)



再看一个复杂的函数：

func4 是一个指针，指向了一个函数(int *),返回值是一个指针，指针类型指向了一个数组[5] 返回值是一个数组指针，数组的元素存放int类型

```c
int (*(*func4)(int *p))[5];
```

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632471926297-c37e433e-4c39-49a6-b191-c119ce5de35c.png)

再来看 开篇的那个复杂的函数

 pfn 是一个指针，指向了一个函数(int *),返回值是一个指针，指针的类型指向了一个数组[10],返回值是一个数组指针，数组的元素存放的int *

```c
    int *(*(*pfn)(int *))[10];
```

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632471975273-5ed287a6-4444-4c22-804d-3d3fdd3a0729.png)

通过上述由易到难解读了复杂函数，以后在遇到复杂函数都可以轻松解读，依据右左法则进行解读。



函数中还需要注意：**函数不能返回数组，而且数组中的元素也不能是函数类型**。一般都会有指针来代替函数的返回数组类型，以及数组元素通过指针指向函数类型。

```c
    //C 语言不能直接返回一个数组，需要通过指针指向数组,返回指针即可
//    int (*func6)(int)[5];
    int (*(*func6)(int))[5];

    //func5 是一个具有5个元素的数组，这个数组的元素都是函数，这是非法的，因为数组的元素除了类型必须一致，每个元素的所占空间也必须相同
    //函数通常所占用的空间是不相同的，一般都是通过指针来指向函数。
//    int func5[5](void);
    int (*func5[5])(void);//指针数组 元素指向的是一个函数
```

### IO 文件操作

C 语言 如何对文件进行操作，这也是需要掌握的技术点之一。

1. 创建文件

C 创建的文件在.exe的同级目录下，因为C文件.c 编译 -> .o文件 -> 编译可执行的.exe文件

```c
//c文件编译：.c -> .o -> .exe
int main1() {
    FILE *p = fopen("./test/m.txt", "w");//打开文件
    //r 只读，w 写入
    if (p) {
        fclose(p);//关闭文件
        //这时候p就是悬空指针
        p = NULL;
        //文件存在
        printf("文件存在");
    } else {
        //文件不存在
        printf("文件不存在");
    }
    return 0;
}
```

1. 读取文件 读取文件的目录必须和.exe文件在同级目录下

```c
/**
 * 读文件
 */
int main2() {
    //读文件
    FILE *p = fopen("./test/m.txt", "r");
    if (p) {
        while (1) {
            char c = fgetc(p);
            //如果读到的是EOF表示读取完毕
            if (c == EOF) {
                break;
            }
            printf("c:%c\n", c);//c:a
        }
    }
    fclose(p);
    p = NULL;
    return 0;
}
```

1. 写文件操作

```c
/**
 * 写文件
 */
int main3() {
    FILE *p = fopen("./test/m.txt", "w");//打开文件
    if (p) {
        fputc('b', p);//全部覆盖重写
        fclose(p);
        p = NULL;
    }
    return 0;
}
```

1. 复制文件的操作

```c
/**
* copy文件
*/
int main() {
    FILE *p = fopen("./test/write.txt", "w");//复制到新文件中
    FILE *p1 = fopen("./test/m.txt", "r");//读取原文件
    if (p == NULL || p1 == NULL) {
        return -1;
    }
    while (1) {
        char c = getc(p1);
        if (c == EOF) {
            break;
        }
        fputc(c, p);
    }
    fclose(p);
    fclose(p1);
    p = NULL;
    p1 = NULL;
    return 0;
}
```

1. 解析文件内容 **重点！！**

在使用C做音视频开发的时候经常要解析音视频文件，这里需要掌握如何解析文件，先来解析一些简单的内容。

例如文件中有如下的内容，进行解析并计算结果

```c
1+2=
4-2=
1*3=
6/2=
```

这里涉及到了通过`fgets`读取一行数据，`sscanf`进行格式化读取,一般单个字符读取通过`EOF`来判断结束，那么如果按照一行读取通过`feof()`函数来进行判断是否读取到最后一行

```c
int acc(int a, char b,int c){
    int result = 0;
    switch (b) {
        case '+':
            result = a + c;
            break;
        case '-':
            result = a - c;
            break;
        case '*':
            result = a * c;
            break;
        case '/':
            result = a / c;
            break;
    }
    return result;
}

int main() {
    FILE *p = fopen("./test/m1.txt", "r");
    if (p) {
        while (1) {
            char buffer[100] = {0};//定义100个char,都设置为0，内存中是有脏数据的 推荐都设置为0
            //读取一行 gets
            fgets(buffer, sizeof(buffer), p);//将一行的数据 读取到buffer中
            int a = 0;
            char b = 0;
            int c = 0;
            //格式化读取
            sscanf(buffer, "%d%c%d", &a, &b, &c);
            printf("a:%d b:%c c:%d = %d\n",a,b,c, acc(a,b,c));

            //判断文件结尾 通过feof判断文件是否读取到了末尾
            if (feof(p)) {
                break;
            }
        }
    }
    fclose(p);
    p=NULL;
    return 0;
}
```

执行的结果如下：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632809427820-b67c9b42-2077-4624-aa9e-74b43ee15d18.png)



下面试着读取视频文件，代码如下：

```c
#define BLOCK_SIZE (1024 * 64)

int main(int argc,char** argv){
    //argc 参数长度 size
    //argv[0] 被系统占用
    //rb 读取二进制  wb 写二进制
    printf("argc:%d agrv1:%s,agrv2:%s\n",argc,argv[1],argv[2]);//argc:1 agrv:(null)
    FILE *pr1 = fopen(argv[1],"rb");
    if(pr1 == NULL) return 0;
    FILE *pr2 = fopen(argv[2],"wb");
    if(pr2 == NULL) return 0;

    //先设置缓冲区
    struct stat st = {0};
    //获取文件的大小
    stat(argv[1],&st);
    int size = st.st_size;//这个大小如何设置内不能说 视频100M 分配100M的内存 一定要进行判断
    printf("size:%d\n",size);
    if(size > BLOCK_SIZE){//如果大于64KB 则分块读取文件
        size = BLOCK_SIZE;
    }
    char *buf = calloc(1,size);//calloc 声明一块内存空间 并且置为内存的初始值  malloc声明一块内存空间 不会置为0

    unsigned int index = 0;//auto 自动正负号 unsigned正数

    while (!feof(pr1)){
        //读取文件 返回最终读取了多少个
        int res = fread(buf,1,size,pr1);//读取pr1文件 读取1个size大小，读取到buffer中去
        //写入目标文件
        fwrite(buf,1,res,pr2);
        index ++;
    }
    free(buf);
    buf = NULL;
    fclose(pr1);
    fclose(pr2);
    pr1 = NULL;
    pr2 = NULL;
    return 0;
}
```

在exe目录下输入一下的命令：`cbase.exe ./test/input.flv ./test/output.flv` 在test文件下存放input.flv文件，output.flv文件是生成的视频文件。

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632812044523-eb5914ae-d0a6-42d9-a93a-9255c98a7580.png)

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1632812027922-cd6159fc-0ad9-4094-b3db-52c0d275c1db.png)



int (*(*(*func)(int *))[5]) (int *);  func 指向函数(int *),函数的返回值是* 指针，指向的是个数组[5],数组的元素类型是*指针，指向了一个函数(int *) 函数的返回值为int



int (* (*func) [5][6])[7] [8]   fun是指针指向一个二维数组[5][6],二维数组的元素类型是一个指针，指向了二维数组[7][8] 元素的类型是int



EOF   feof() 可以判断文件读取结束