# C 语言基础快速入门

### 认识C文件的代码结构

```c
#include <stdio.h>
//声明文件 .h .hpp
//<> 表示寻找系统的资源
//"" 表示寻找开发者定义的声明文件
//.c .cpp 实现文件
//代码结构
//main 函数只能存在一个
int mainT1() { //函数的入口
    printf("你好，Hello, World!\n");
//    getchar();// 阻塞 用户输入

    return 0;
}
```

### 基本数据类型

```c
//
// Created by JakePrim on 2021/4/8.
// 基本数据类型
//
#include <stdio.h>

int mainT2() {
    int i = 100;
    double d = 200;
    float f = 200;
    long l = 100;
    short s = 100;
    char c = 'd';
    //字符串
    char *str = "JakePrim";

    //打印需要占位
    printf("i的值是%d\n", i);//d == 整形
    printf("d的值是%lf\n", d);//lf == long float 双精度
    printf("f的值是%f\n", f);//f == float
    printf("l的值是%d\n", l);//d == 整形
    printf("s的值是%d\n", s);//s == short
    printf("c的值是%c\n", c);//c == char

    printf("字符串:%s\n", str);//JakePrim

    return 0;
}
```

### 占用字节数

```c
//
// Created by JakePrim on 2021/4/8.
//
#include <stdio.h>

//基本类型占用的字节数 sizeof 判断占用的字节数
int mainT3() {
    printf("int 占用的字节数:%lu\n", sizeof(int));//4
    printf("double 占用的字节数:%lu\n", sizeof(double));//8
    printf("char 占用的字节数:%lu\n", sizeof(char));//1
    return 0;//NULL == 0
}
```

### 初识指针

指针存储的永远都是内存地址

- 每个变量都会为其分配一个地址内存地址，用来存储变量的值
- 指针存储的是内存地址
- 指针变量也有地址的，和指针变量赋值的地址是不一样的
- 指针占用的字节就是固定的8个字节，指针的大小是不变的和指针的类型无关，地址是没有长短之分的 地址是固定的长度
- 如果只声明指针变量，没有赋值就是野指针（int *p;）
- 指针是有类型的，不能char类型的指针，存储int类型的指针

```c
//
// Created by JakePrim on 2021/4/8.
//
#include <stdio.h> //stdio 标准库

// C 万物皆指针
// Linux 万物皆文件
int mainT4() {
    //指针 == 地址

    //例如 基本变量存储在栈中，栈也会由编译器分配一个地址来存储变量的值
    int number = 10000;
    // %p 输出地址的占位
    // & == 取址
    printf("number在内存中的地址:%p\n", &number);//0x7ffeeea5ff08

    //任何变量都是地址 使用地址获取值
    // * == 取出地址存储的值
    printf("获取number地址的值:%d\n", *(&number));//10000

    //指针永远存放的是内存地址
    int *intp = &number;
    printf("获取指针变量：%p\n", intp);//指针变量 存储的永远是内存地址 0x7ffeec070f08
    printf("获取指针变量内存地址的值:%d\n", *intp);//获取地址的值 10000

    //int * 表示int类型的指针
    //内存地址就是指针，指针就是内存地址
    //intp 指针别名、指针变量
    *intp = 300;//修改了&number地址存储的值，那么number的值也会修改
    printf("number：%d\n", number);//300
    
    int *p;//指针存储的是内存地址,只能是地址,指向内存的某一个区域 计算机中有地址总线
    //如果只声明指针，不给指针赋值就是一个野指针
    int a = 10;//实体变量
    p = &a;//指针p赋值为a的内存地址
    //查看a的地址
    printf("&a = %p\n",&a);//&a = 000000000061FDE4
    //查看指针变量p存储的内容
    printf("p=%p\n",p);//p=000000000061FDE4
    //指针变量p也是有地址的,*p取出改地址的实体也就是存储的值
    printf("p=%p,*p=%d\n", &p, *p);//p=000000000061FDE8,*p=10

    printf("sizeof p = %llu\n",sizeof p);//查看指针的大小sizeof p = 8

    char b = 'c';
    char *c = &b;
    char d = *c;//取出指针变量c，存储地址的实际值
    printf("sizeof c = %llu\n",sizeof c);//sizeof c = 8
    //指针占用的字节就是固定的8个字节(64为8个字节，32位4个字节)，指针的大小是不变的和指针的类型无关，地址是没有长短之分的 地址是固定的长度
    //注意：指针是有类型的，不能char类型的指针，存储int类型的指针

    printf("d=%c\n",d);//d=c

    return 0;
}
```

### 声明函数

函数不能写在main()的下面

```c
//
// Created by JakePrim on 2021/4/9.
//
#include <stdio.h>

//函数 在C语言中函数必须放在main函数的前面
//void change(){
//    printf("123");
//}
// 或者先声明在使用
void change(int i);

//C语言不允许函数重载的 Java和C++是可以的
void change2(int *i);

/**
 * 交换两个变量的值
 * @param a
 * @param b
 */
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main() {
    int i = 100;
    change(i);
    printf("%d，地址:%p\n", i, &i);//100 0x7ffee369df08 i的值并没有改变 因为和change的形参i的地址是不一样的
    change2(&i);
    printf("%d，地址:%p\n", i, &i);//100 0x7ffee369df08 i的值改变了
    int a = 10;
    int b = 11;
    swap(&a, &b);
    printf("交换后的结果:%d,%d\n", a, b);//11,10
    return 0;
}

//根据声明实现函数
void change(int i) {
    printf("change->i:%p\n", &i); //0x7ffee369deec
    i = 200;
}

//使用指针
void change2(int *i) {
    printf("change2->i:%p\n", i); //0x7ffeee314f08 传入的是main中i的地址
    *i = 200;//修改地址中的值
}
```

### 多级指针

```c
//
// Created by JakePrim on 2021/4/9.
//
#include <stdio.h>

//多级指针
int mainT6() {
    int num = 100;

    int *num_p = &num;//num的内存地址

    int **num_p_p = &num_p;//num_p的内存地址

    int ***num_p_p_p = &num_p_p;//三级指针

    printf("num_p：%p,num_p_p:%p\n", num_p, num_p_p);//num_p：0x7ffeeb580f08,num_p_p:0x7ffeeb580f00

    printf("num_p:%p\n", *num_p_p);//取出num_p_p存储的值 num_p:0x7ffeeb580f08

    printf("num:%d\n", **num_p_p);//num:100  *num_p_p == &num   **num_p_p = num

    //多级指针的意义

    return 0;
}
```

### 数组与指针

```c
//
// Created by JakePrim on 2021/4/9.
//
#include <stdio.h>

//数组与指针
int mainT7() {

    int arr[] = {1, 2, 3, 4};

    //如下遍历数组在其他平台上会报错
//    for (int i = 0; i < 4; ++i) {
//
//    }

    //规范的写法 在其他平台没有问题的
    int i = 0;
    for (i = 0; i < 4; i++) {
        printf("%d\n", arr[i]);
    }
    //数组的内存地址 == 第一个元素的内存地址
    printf("arr == %p\n", arr);//arr == 0x7ffeed6d6ef0
    printf("&arr == %p\n", &arr);//&arr == 0x7ffeed6d6ef0
    printf("&arr[0] == %p\n", &arr[0]);//&arr[0] == 0x7ffeed6d6ef0
    printf("%d\n", *(arr));//1 第一个元素

    //将数组的内存地址赋值给指针
    int *arr_p = arr;

    //使用指针遍历数组
    for (i = 0; i < 4; i++) {
        printf("%d\n", *(arr_p + i));
        printf("%p\n", arr_p + i);//内存地址会相差4，因为int占用4个字节
    }
    printf("*arr_p:%d\n", *arr_p);//由于arr的内存地址指向了第一个元素，所以取值就是第一个元素 1
    //指针位移，由于数组内存地址是连续的所以指针+1就是第二个元素
    arr_p++;
    printf("*arr_p:%d\n", *arr_p);//取值就是第一个元素 2


    return 0;
}

//
// Created by JakePrim on 2021/4/10.
//
#include <stdio.h>

int mainT8() {
    int arr[4];

    int *arr_p = arr;//指向数组元素的第一个元素的地址

    for (int i = 0; i < sizeof arr / sizeof(int); ++i) {
        *(arr_p + i) = i + 1000;//通过指针给数组赋值
    }

    //sizeof arr = arr 是有4个int，也就是16个字节，所以计算数组的长度就是数组的字节数/类型的字节数
    for (int i = 0; i < sizeof arr / sizeof(int); ++i) {
        //两种取数组值的方式
        printf("%d\n", *(arr_p + i));
        printf("%d\n", arr_p[i]);
    }

    //指针占用的内存大小是4个字节
    printf("指针占用的字节大小:%d\n", sizeof(arr_p));//64位：8字节   32位：4个字节
    return 0;
}
```

### 函数指针

```c
//
// Created by JakePrim on 2021/4/10.
//

#include <stdio.h>

void add(int num1, int num2) {
    printf("num1 + num2 = %d\n", (num1 + num2));
}

void sub(int num1, int num2) {
    printf("num1 - num2 = %d\n", (num1 - num2));
}

//操作回调
//void(*method)(int, int) -> 声明的函数指针
//void -> 返回值
//(*method) -> 函数名
//(int,int) -> 两个参数
void operate(void(*method)(int, int), int num1, int num2) {
    method(num1, num2);
}

void callBackMethod(char *fileName, int current, int total) {
    printf("压缩的文件:%s,进度：%d/%d", fileName, current, total);
}

void compress(char *fileName, void(*callback)(char *, int, int)) {
    //开始压缩图片 压缩进度回调
    callback(fileName, 5, 100);
}

//函数指针 - 相当于java接口的回调
int mainT9() {
    //函数add就是在内存中的地址 add和&add是一样的
    printf("add的地址:%p\n", add);//add的地址:0x1011a0be0
    printf("&add的地址:%p\n", &add);//&add的地址:0x1011a0be0

    operate(add, 1, 2);//num1 + num2 = 3

    operate(sub, 2, 1);//num1 - num2 = 1

//    void (*call)(char *, int, int) = callBackMethod;

    //兼容性问题，先声明在使用
    void (*call)(char *, int, int);//定义函数指针
    call = callBackMethod;//给函数指针赋值

    compress("1.png", call);

    return 0;
}
```

### C 函数库

```c
//
// Created by JakePrim on 2021/4/11.
//
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>

//C函数的使用
int mainT10() {
    //初始化随机数发生器函数
    srand((unsigned) time(NULL));

    int i;
    for (i = 0; i < 10; ++i) {
        printf("随机数:%d\n", rand() % 100);//rand()获取随机数
    }

    //字符串拷贝函数
    char str_arr[10];
    char *str = "abcdefg";
    strcpy(str_arr, str);// 将str拷贝到str_arr中
    printf("%s\n", str_arr);//abcdefg

    return (0);//== return 0
}
```

### 静态/动态开辟内存

```c
// 动态开辟: malloc 在堆区开辟 内存空间 动态范畴

#include <stdio.h>
#include <stdlib.h>


//C 语言在开发中不能出现野指针和悬空指针
void dynamicAction() {
    int *p;//野指针 没有具体的指向

    //申请在堆中开辟一段内存区域 返回void *：表示可以转换成任意类型int *   double *....
    int *arr = (int *) malloc(1 * 1024 * 1024);//申请堆区大小是1M

    printf("arr自己的内存地址:%p,堆区的内存地址：%p\n", &arr, arr);//arr自己的内存地址:0x7ffee57faee8,堆区的内存地址：0x7fe219500000
    free(arr);//释放申请的堆区
    //释放堆区后，arr还是指向了堆区的内存地址，这就会有悬空指针的问题
    printf("arr自己的内存地址:%p\n", arr);//arr自己的内存地址:0x7fe219500000
    arr = NULL;//不设置null会可能为悬空指针 ，需要重新指向一块内存地址，因为free，arr还是指向了堆区的地址
    printf("arr自己的内存地址:%p\n", arr);//arr自己的内存地址:0x0
    //malloc 申请堆区内存  free释放堆区
}//函数弹栈后 不会自动释放堆区 如果不及时释放 会导致越来越大

/**
 * 什么时候才会使用动态开辟呢？使用场景？
 * 静态开辟的内存空间大小是不能改变的
 */
int main() {
    for (int i = 0; i < 2; ++i) {
        dynamicAction();
        //arr自己的内存地址:0x7ffeebe24ee8,堆区的内存地址：0x7fe6a7e00000
        //arr自己的内存地址:0x7ffeebe24ee8,堆区的内存地址：0x7fe6a7e00000
    }

    int arr[6];//静态开辟 空间大小是不能改变的，静态开辟会自动回收安全可靠

    //开辟的空间想要变化 需要使用动态开辟
    int num;
    printf("请输入个数:");
    //获取用户输入的值
    scanf("%d", &num);
    //动态开辟
    int *arr2 = (int *) malloc(sizeof(int) * num);

    int result;
    for (int i = 0; i < num; ++i) {
        printf("请输入第%d\n", i);
        scanf("%d", &result);
        arr2[i] = result;
    }

    //改变内存的大小
    int *new_arr = (int *) realloc(arr2, num * 2);
    //new_arr != null 表示开辟成功
    if (new_arr) {// == new_arr != NULL
        for (int i = 0; i < num; ++i) {
            printf("得到新的内存：%d\n", new_arr[i]);
        }
    }

    //改变内存大小的地址：0x7fbd8e704080,原有的内存地址:0x7fbd8e704080 是同一个地址
    printf("改变内存大小的地址：%p,原有的内存地址:%p\n", new_arr, arr2);

    
    if(new_arr){
    	free(new_arr);
    	new_arr = NULL;
    }
    //原有的指针也需要置为NULL 否则成为悬空指针
    //改变内存大小的地址：0x0,原有的内存地址:0x7fe16a4058d0
    printf("改变内存大小的地址：%p,原有的内存地址:%p\n", new_arr, arr2);

    return 0;
}
```

### 字符串常见操作

```c
/**
 * 把字符串全部转换为小写
 * @param dest
 * @param name
 */
void lower(char *dest,char *name){
    //不要破坏name指针的地址，否则外部的也会改变
    char *temp = name;
    while (*temp){
        //一个个字符的转换
        *dest = tolower(*temp);
        //挪动指针
        temp++;
        dest++;
    }
    //dest指针的地址指向了最后
    *dest = '\0';//避免打印系统值
}

/**
 * 截取字符串
 * @param result 截取的结果
 * @param dest 目标字符串
 * @param start 开始的位置
 * @param end 结束的位置
 */
void cutout(char *result,char *dest,int start,int end){
    if (start > end){
        *result = '\0';
    }
    //不要破坏dest
    char * temp = dest;
    temp+=start;
    for (int i = start; i < end; ++i) {
        *result = *temp;
        result++;
        temp++;
    }
    *result = '\0';
}
int main(){
    //通过数组创建字符串
    char str[] = {'H','e','l','l','o','\0'};
    str[2] = 'r';
    //注意printf 必须遇到\0才会结束否则会有乱码问题，在数组的末尾加上\0
    printf("第一种方式:%s\n",str);//第一种方式:Hello烫烫烫蹄鼭L?╔s?

    //通过指针创建字符串
    char * str2 = "Hello";// 默认会在末尾加上\0结束符号
    str2[2] = 'r';//这里会崩溃?? 不允许访问 但是IDE没有问题呢？ 因为str2指向的是一个地址
    printf("第二种方式:%s\n",str2);

    return 0;
}
```

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1620720153186-b182995c-c058-4416-ac91-e66ffb9a477f.png)

截取字符串操作:

```c
/**
 * 第三种方式
 * @param result
 * @param dest
 * @param start
 * @param end
 */
void cutout3(char *result,char *dest,int start,int end){
    //简化写法
    for (int i = start; i < end; ++i) {
        *(result++) = *(dest+i);
    }
}

/**
 * 截取字符串
 * @param result 截取的结果
 * @param dest 目标字符串
 * @param start 开始的位置
 * @param end 结束的位置
 */
void cutout(char *result,char *dest,int start,int end){
    if (start > end){
        *result = '\0';
    }
    //不要破坏dest
    char * temp = dest;
    temp+=start;//直接讲地址移动到开始的位置
    for (int i = start; i < end; ++i) {
        *result = *temp;
        result++;
        temp++;
    }
    *result = '\0';
}

/**
 * 第四种方式
 */
void cutout4(char *result,char *dest,int start,int end){
    //参数1 赋值到容器中
    //参数2 直接从start的位置开始
    //参数3 copy的个数
    strncpy(result,dest+start,(end-start));
}
```



### 结构体

- 声明结构体，及结构体赋值

```c
/**
 * 结构体就相当于Java中的类的概念
 * @return
 */
struct Dog{
    //定义成员
    char name[10];
    char *n;//可以直接赋值
    int age;
    char sex;
} g1 = {"gs","g",2,'M'},g2={"g3","g",4,'G'},g3;//结尾必须有;结束符号


int mainT2(){
    //使用结构体
    struct Dog dog;//这样写完是没有任务初始化的 成员默认值是系统值(乱码)
    //赋值操作
    //注意数组赋值 需要使用strcpy api函数
    strcpy(dog.name,"旺财");
    dog.age = 3;
    dog.sex = 'G';

    printf("name:%s,age:%d,sex:%c\n",dog.name,dog.age,dog.sex);//name:旺财,age:3,sex:G

    //直接使用
    g3.n = "123";//指针不用使用strcpy 进行赋值
    printf("name:%s,age:%d,sex:%c\n",g1.name,g1.age,g1.sex);//name:gs,age:2,sex:M

    return 0;
}
```

- 结构体嵌套

```c
struct Study{
    char *studyContent;//学习内容
};
//结构体嵌套
struct Student{
    char name[10];
    int age;
    char sex;
    struct Study study;//Clion工具的写法
//    Study st;vs的写法
    struct Wan{
        char * wanContent;
    } wan;//定义结构体别名
};

int mainT3(){
    //使用结构体
    struct Student student = {"丽丽",18,'N',
            {"学习C语言"},
            {"游戏"}
    };
    //获取嵌套的结构体
    student.study.studyContent;
    student.wan.wanContent;
    printf("姓名：%s",student.name);
    return 0;
}
```

- 结构体指针

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <malloc.h>
struct Cat{
    char name[10];
    int age;
};
int mainT4(){
    //栈区
    //声明结构体
    struct Cat cat = {"猫马上",2};
    //结构体指针
    //vs的写法：Cat *catp = &cat;
    struct Cat * catp = &cat;
    //-> 调用一级指针成员
    catp->age = 18;
    strcpy(catp->name,"毛毛");
    printf("name:%s,age:%d\n",catp->name,catp->age);


    //堆区 动态开辟
    //申请空间
    struct Cat *c1 = (struct Cat *)malloc(sizeof(struct Cat));
    strcpy(c1->name,"213");
    c1->age = 19;
    printf("name:%s,age:%d\n",c1->name,c1->age);
    //释放空间
    free(c1);
    c1 = NULL;


    return 0;
}
```

- 结构体数组

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
struct Cat3{
    char name[10];
    int age;
};
int mainT5(){
    //结构体数组
    struct Cat3 cat3[10]={
            {"xs",1},
            {"xs2",2},
            {"xs3",3},
    };

    struct Cat3 tc9 = {"s9",9};
//    cat3[9] = tc9;
    //指针 和上面是等价 的
    *(cat3 + 9) = tc9;
    printf("name:%s,age:%d\n",cat3[9].name,cat3[9].age);

    //动态开辟
    struct Cat3 *cat2 = malloc(sizeof(struct Cat3) * 10);//开辟10个Cat3的空间大小 和上面的数组[10]是一样的

    //赋值操作 cat2默认指向首元素地址
    strcpy(cat2->name,"12321");
    cat2->age = 10;
    printf("name:%s,age:%d\n",cat2->name,cat2->age);//name:12321,age:10

    //给第二个元素赋值
    struct Cat3 *temp = cat2;
    temp+=1;
    strcpy(temp->name,"2222");
    temp->age = 11;
    printf("name:%s,age:%d\n",temp->name,temp->age);//name:2222,age:11

    printf("cat:%p\n",cat2);//006B58F8
    printf("temp:%p\n",temp);//006B58F8
    //注意在释放时需要回到首元素 因此尽量不去改动cat2的指向而是通过temp
    free(cat2);
    cat2 = NULL;
    printf("cat free之后\n");
    printf("cat:%p\n",cat2);//00000000
    printf("temp:%p\n",temp);//006B58F8 注意temp成了悬空指针 需要置为NULL
    temp = NULL;

    return 0;
}
```

- typedef 定义结构体类型 抹平不同工具平台的差异化

```c
#include <stdio.h>
#include <stdlib.h>

//在学习结构体的时候会发现在VS 和 Clion平台的写法不一致那么如何避免这种问题呢？
//定义结构体类型
struct Worker_{
    char *name;
    int age;
    char sex;
};
//VS 的写法typedef Worker_
//定义typedef 可以兼容代码写法
typedef struct Worker_ Worker_;//给结构体取别名

typedef struct Worker_ * Worker;//给结构体指针取别名

//匿名结构体的别名 推荐使用给结构体取了别名 = AV
typedef struct {
    char *name;
    int age;
    char sex;
} AV;

//在方法中使用结构体
void show(AV* a){

}



int mainT6(){
    //两种平台的写法就会一致
    Worker worker = malloc(sizeof(Worker_));

    //不用考虑差异化
    AV * av = malloc(sizeof(AV));

    AV a = {"测试",123,'N'};

    return 0;
}
```

### 枚举

```c
#include <stdio.h>
//枚举 int类型的
enum CommonType{
    TEXT = 10,//不给值默认是从0开始
    TEXT_IMAGE,
    IMAGE
};

//定义别名 抹平不同工具的差异化
typedef enum CommonType Common;

//推荐写法 注意注意
typedef enum{
	AUDIO = 2,
    IFNO,
    VIDEO
} AV;

int main(){
    //clion写法
    enum CommonType commonType = TEXT;
    //VS工具的写法
    //CommonType commonType = TEXT;

    //抹平差异化
    Common common = TEXT;
    Common common2 = TEXT_IMAGE;
    Common common3 = IMAGE;

    printf("common:%d,common2:%d,common3:%d\n",common,common2,common3);//common:10,common2:11,common3:12



    return 0;
}
```

### 文件操作

- 文件读取操作

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int main(){
    //文件操作
    //参数一：_Filename 文件的来源
    //参数二：_Mode 模式r(读) w(写)  rb(作为二进制文件的读) wb(作为二进制文件的写)
    //返回值：FILE 结构体
//    fopen();
    printf("读取文件\n");

    char *fileName = "D:\\ee.txt";//文件的路径
    FILE * file = fopen(fileName,"r");
    if(!file){
        //如果file为null就表示读取失败了
        printf("文件打开失败");
        exit(0);//退出程序
    }
    printf("读取文件成功\n");
    //读取文件的内容，先定义缓冲区
    char buffer[10];
    //fgets:1:缓冲区 2：长度10 3 ：文件
    while (fgets(buffer,10,file)){
        printf("%s",buffer);
    }

    //关闭文件
    fclose(file);

    //文件的写操作
    char *fileName2 = "D:\\ee2.txt";//文件的路径
    FILE * wfile = fopen(fileName2,"w");
    if(!wfile){
        printf("打开文件失败");
        exit(0);
    }
    fputs("写入文件",wfile);

    //关闭文件
    fclose(wfile);

    return 0;
}
```

- 文件复制操作

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

//文件的复制 二进制文件来复制
int main(){
    char *fileName = "D:\\ee.txt";//copy的来源
    char *fileName2 = "D:\\ee2.txt";//copy的目标

    FILE * file = fopen(fileName,"rb");//二进制读取
    FILE * wfile = fopen(fileName2,"wb");//二进制写入
    if(!file || !wfile){
        printf("文件打开失败");
        exit(0);
    }
    //获取文件的大小
    //移动头指针到文件的末尾
    //SEEK_CUR 代表是当前的位置
    //SEEK_SET 代表是开头的位置
    //SEEK_END 代表是结尾的位置
    fseek(file,SEEK_SET,SEEK_END);
    //会给file指针赋值，挪动的记录
    long file_size = ftell(file);//读取刚刚给file记录的信息
    printf("%s,文件的大小:%ld\n",fileName,file_size);//D:\ee.txt,文件的大小:16

    int buffer[512];//512 * 4 = 2048个字节
    int len;//每次读取的长度
    //读取目标文件操作:
    //参数1：读取到容器中去
    //参数2：每次偏移多少
    //参数3：容器大小 不要直接写数字：sizeof(buffer)/ sizeof(int) = 512
    //参数4：目标文件
    while ((len= fread(buffer, sizeof(int), sizeof(buffer)/ sizeof(int),file))!=0){
        //写入到目标文件
        fwrite(buffer, sizeof(int),len,wfile);
    }

    //关闭文件
    fclose(file);
    fclose(wfile);

    return 0;
}
```





- 文件加密及解密

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

//使用密钥加密
int main(){
    char * fileNameStr = "D:\\1.png";//要加密的文件
    char * fileNameStrEncode= "D:\\1encode.png";//加密后的文件

    //读取要加密的文件
    FILE * file = fopen(fileNameStr,"rb");
    //写入加密后的文件
    FILE * fileEncode = fopen(fileNameStrEncode,"wb");//wb 会自动厂家按文件

    //密钥
    char * password = "123456";

    if(!file || !fileEncode){
        printf("文件打开失败");
        exit(0);
    }
    int c;
    int index = 0;
    int pass_len = strlen(password);
    while ((c= fgetc(file))!=EOF){
        char item = password[index%pass_len];
        printf("item=%c\n",item);
        fputc(c ^ item,fileEncode);
        index++;
    }
    fclose(file);
    fclose(fileEncode);
    return 0;
}


#include <stdio.h>
#include <stdlib.h>
#include <string.h>

//解密文件
int main(){
    char * fileNameStrEncode= "D:\\1encode.png";//加密的文件
    char * fileNameStrDecode= "D:\\1decode.png";//解密后文件

    FILE * encodeFile = fopen(fileNameStrEncode,"rb");
    FILE * decodeFile = fopen(fileNameStrDecode,"wb");

    if(!encodeFile || !decodeFile){
        printf("文件打开失败");
        exit(0);
    }
    //加密文件：把每一个字节拿出来，对每一个字节进行处理(全部加密)。把某一部分内容拿出来处理(部分加密)
    //异或处理 10 ^ 5 = 11111
    //解密：还原 1111 ^ 5 = 10
    //密钥
    char * password = "123456";
    int c;//接受读取的值
    int index = 0;
    int pass_len = strlen(password);
    //fgetc 返回值EOF=-1 就会没有了false
    while ((c= fgetc(encodeFile)) != EOF){
        char item = password[index%pass_len];
        printf("item=%c\n",item);
        //加密操作 对c进行异或操作
        fputc(c^item,decodeFile);
        index++;
    }
    //关闭文件
    fclose(encodeFile);
    fclose(decodeFile);
    return 0;
}
```