# C++ | 快速入门

在前面的几篇文章中重点讲解了C语言，而对于C++语言，只要一篇文章即可，C++是对C语言的扩展，基于面向对象的编程，掌握了C就等于掌握了一般的C++，只要有面向对象语言基础的都可以理解C++，主要讲解C++独有的一些特性。

### C++的代码结构

```cpp
#include <iostream> //C++的标准支持

using namespace std;//命名空间 c++的特性 类似Java的内部类

int mainT() {
    //C++可以运行C语言的
    printf("C版\n");//C语言的打印
    //:: 所在的作用域
    //std::cout << "Hello, World!" << std::endl;
    //在前面引入了命名空间可以省去std::
    cout << "Hello, World!" << endl;//C++中的打印  endl == \n 换行
    //<< 是操作符重载 kotlin也有操作符重载
    cout << "收到\n" << "四大皆空\n";
    const int number = 100;
    cout << number << endl;
    return 0;
}
```

### 需要注意的运算符

* sizeof 返回变量的大小
* 条件运算符 condition ? X : Y
* 逗号运算符 
* 强制运算符，数据类型转换 int(2.22) = 2 不建议这样使用
* . -> 成员运算符：用于引用类、结构体和共用体
* & 指针运算符：返回变量的地址
* *指针运算符：指向一个变量

```cpp
void OtherFun() {
    int A = 10;
    int B = 20;
    cout << sizeof(A) << endl;//4
    int C = A > B ? 1 : 0;
    cout << C << endl;//0
    int E = (A, B, C);
    cout << E << endl;//0
    float F = float(E);
    cout << F << endl;//0
    cout << &F << endl;//0000000DD9AFF8C4
    float* P = &F;
    cout << P << endl;//0000000DD9AFF8C4
    cout << *P << endl;//0
    typedef struct Week{
        short Sunday = 0;
        short Monday = 1;
        short Tuesday = 2;
        short Wednesday = 3;
        short Thursday = 4;
        short Friday = 5;
        short Sturday = 6;
    } Week;
    Week w;
    cout << w.Friday << endl;//5
    cout << w.Monday << endl;//1
    cout << sizeof(w) << endl;//14 (2 * 7 = 14)
}
```

* ^ 两个值不同 是真，否则为假
* & 两个值都为1 是真 否则为假
* | 一个值为真 是真，都为假 是假
* p = 0 q = 1 ; q & p = 0  q | p = 1 q ^ p = 1
* p = 1 q = 1; q & p = 1  q | p = 1 q ^ p = 0

```cpp
	int A = 10;
	int B = 20;
	cout << (A & B) << endl;//01010 & 10100 = 00000 = 0
	cout << (A | B) << endl;//01010 & 10100 = 11110 = 30
	cout << (A ^ B) << endl;//01010 & 10100 = 11110 = 30
	cout << (~A) << endl;
	cout << (A << 2) << endl;
	cout << (A >> 2) << endl;
```

### 引用的原理与常量引用

C++中提出了引用的概念，其实在Java中也经常使用到引用，只不过Java将引用很隐晦的表示不想C++中需要显示的声明引用。

引用的使用如下：

通过&声明引用，引用不会占用内存空间，它只是相当于变量的别名，可以理解为它就是引用的变量，变量就是它，相当于曹操 字孟德，曹操和孟德都是一个人。引用非常常用，它不会占用任何内存空间，直指变量本身，操作引用就是操作变量

```cpp
	int a = 1;
    int &b = a;//&：注意在等号的左边就是表示引用，在等号的右边 int *b = &a;就表示取址 含义是不同的

    printf("a:%d,b:%d\n", a, b);//a:1,b:1
    b = 2;//操作别名就相当于操作a本省
    printf("a:%d,b:%d\n", a, b);//a:2,b:2
    //a 和 b的地址是完全一致的
    printf("a:%p,b:%p\n", &a, &b);//a:000000000061fe14,b:000000000061fe14
```

关于引用的一些特性：

- & 只是起到标识的作用
- 声明引用时，必须同时对其进行初始化
- 声明引用的类型和变量的类型要一致
- 引用声明完毕后，引用名不能作为其他变量名的别名
- 声明一个引用，不是新定义一个变量，它只表示该引用名是目标变量名的一个别名，**本身不是一种数据类型**，不占用存储单元
- 不能建立数组的引用，因为数组是若干元素的集合，**无法建立一个数组的别名**` int aaa[10] = {1,2};int &aa1[10] = aaa;//错误的`

- 引用作为函数的参数：

在C语言中要实现两个数的交换就需要使用指针，进行频繁的取址和取值操作

```c
/**
 * 取地址方式的互换
 * @param number1 地址
 * @param number2 地址
 */
void numberChange(int *number1,int *number2){
    int temp = 0;
    temp = *number1;
    *number1 = *number2;
    *number2 = temp;
}
```

在C++中就可以使用引用进行两个数的交换，引用不会占用内存，直接操作变量本身

```cpp
//
// C++的引用与常量引用
//
#include <iostream>

using namespace std;

/**
 * C++提倡的引用
 */
void numberChange2(int &number1,int &number2){
    cout << "n1地址:" << &number1 << " n2地址:" << &number2 << endl;//n1地址:0x62fe14 n2地址:0x62fe10
    int temp = 0;
    temp = number1;
    number1 = number2;
    number2 = temp;
}

int main(){
    int number = 100;
    int number2 = 200;
    cout << "n1地址:" << &number << " n2地址:" << &number2 << endl;//n1地址:0x62fe14 n2地址:0x62fe10
    numberChange2(number,number2);
    cout << "n1:" << number << " n2:" << number2 << endl;
    return 0;
}
```

值传递，内存地址不一样

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1621581140057-0b12c83d-f490-453f-8dfb-f46c08a963ad.png)



引用，内部地址一样，只是取了不同的别名

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1621581345571-d3641cc3-809a-469e-9d7b-2da87693865e.png)

- 常量引用

给引用添加`const` 关键字就表明是个常量引用 表示不允许通过引用名去修改变量的值。注意：变量被修改后，常量引用的值和变量的值一致，因为不管是常量引用还是引用都和目标变量的内存地址一样，自然值也是一样

```cpp
    //常量引用
    int p = 11;
    const int &pr = p;//常量引用,引用的变量不能通过别名去修改
    //pr = 22;//常量引用：不允许引用/别名去修改变量的值
    p = 22;//当p被修改，那么pr也会能到最新的值，因为引用就是变量的本身
    cout << p << pr << endl;//p=22 pr=22
```

- 引用作为函数的返回值的问题

不能返回局部变量的引用，因为函数的局部变量会在函数返回后就被销毁，被返回的引用就会出现错误

不能返回函数内部new分配的内存引用

如下代码，这样使用是会报错

```cpp
float &fun1(float r) {
    float temp = (float) (r * r * 3.14);
    //和下面的代码一样
//    float &b = temp;
    return temp;
}
//    float f = fun1(10.0);//fun1返回引用，但是返回的是局部的变量的别名/引用，但是函数执行完毕 局部变量temp不在了，
    // 这里要注意 如果函数返回值要返回引用，那么引用的变量不能是函数的局部变量
//    cout << "f=" << f <<endl;

//    float &f = swap();//不能对函数的返回值的局部变量做引用
```

常见的使用方式如下：函数返回的引用是全局变量，直接可以使用

```cpp
int vals[10];

int error = -1;

/**
 * 全局变量作为引用 返回
 * @param n
 * @return
 */
int &put(int n) {
    if (n >= 0 && n <= 9) {
        return vals[n];
    }
    return error;
}

    //常见的引用应用方式
    put(0) = 0;//等价于 put(0) 相当于vals[0]的引用，那么put(0) = 0 就等价于 vals[0] = 0 因为vals是全局变量 引用还存在着
    put(1) = 1;//同样的原理 vals[1] = 1
    cout << vals[0] << vals[1] << endl;
```

### #define、const和constexpr

- `#define`是**预处理阶段**进行处理，是一种宏定义，所以定义常量的宏是没有类型的，是在编译前即预编译阶段进行字符替换，并且由于是在预处理阶段替换所以不会有类型安全检查，系统也不会为它分配内存，存储在程序的代码段空间，实际就是给出了立即数，**在运行过程中，常量在内存中会有若干个拷贝**；
- `const`是一种Runtime，`const`常量会在内存中分配，可以使堆中也可以是栈中。以后在定义的常量调用时，只是使用对应的内存地址，不再开辟新的空间，在内存中只有一个拷贝，因此`const`相比`#define`，可以避免反复分配内存，节省空间;
- `constexpr`与`const`一样，它可以应用于变量，不同的是可以应用于函数和类构造函数,`constexpr`指示值或返回值是常量，并且在可能的情况下，在编译时计算

```cpp
constexpr auto ASPECT_RATIO = 1.653;
#define ASPECT_B = 1.564
const double ASPECT_R = 1.653;//常量
```

### 内联函数

C ++中的内联函数，主要用来解决C语言中的预处理宏中函数的问题，内联函数为了继承宏函数的效率，没有函数调用时的开销，`inline `内联的关键字，实现更加简单和函数的方式一致，代替了#define的繁琐操作保持了预处理宏的功能，更加安全可靠，可以定义复杂的函数。将函数体的代码插入到调用的地方适用于功能简单，功能较小 。不能有循环体 Switch 递归等不能使用内联函数，否则会自动将`inline`去掉 变成普通函数.

**内联函数是不能取址的，没有意义**



```cpp
inline int add(int x,int y){
    return x + y;
}

int main(){
    int res = add(1,2);
    cout << res << endl;
    return 0;
}
```

内联函数的作用：在编译时将函数体嵌入到每一个调用处，适用于功能简单，规模较小又使用频繁的函数。



内联函数的缺点：

- 内联函数具有一定的局限性，内联函数的函数体一般来说不能太大，如果内联函数的函数体 过大，一般的编译器会放弃内联的方式，采用普通函数的方式调用函数
- inline对编译器来说只是一种建议，编译器可以选择忽略这个建议，一个inline函数不一定会被编译器识别为内联函数



内联函数和宏的区别：

- 宏是由预处理器对宏进行代替，而内联函数是通过编译器控制来实现的
- 内联函数是真正的函数，只是在需要用到的时候，内联函数像宏一样的展开，所以取消了函数的参数压栈，减少了调用的开销
- 内联函数由于宏定义，因为内联函数遵循的类型和作用域规则，它与一般函数更相近
- 宏定义只是简单的文本替换，并没有严格的参数检查，不能享受C++编译器的严格类型检查的好处，存在着一系列的隐患和局限性



### 函数重载

在学习C语言的时候，是不允许函数重载的C++和Java类似是可以函数重载。

```cpp
#include <iostream>
#include <string.h>

using namespace std;

//重载add函数
/**
 * 默认形参赋值 优先寻找
 * void test(int x, int y = 10,int d) 错误的  实参一定要在默认参数前面
 * @param number1
 * @param number2
 * @param isOk 默认参数
 * @return
 */
int add(int number1=100,int number2=200,bool isOk = 0){
    return number1 + number2;
}

int add(int number1,int number2){
    return number1 + number2;
}

int add(int number1,int number2,int number3){
    return number1 + number2 + number3;
}
//函数重载
int main(){
    add(1);
//    add(1,2);

    return 0;
}
```

特殊的函数重载：

1. 变量和引用算不算重载？

引用不管是正常的引用还是常量引用，都和变量冲突，但是引用和常量引用不会冲突

```cpp
void t(int a) {
    cout << "a:" << a << endl;
}
/**
 * 引用算重载函数会和int a冲突 重载的前提具体调用的哪个类型
 */
void t(int &a) {
    cout << "int &a:" << a << endl;
}

/**
 * 常量引用，算重载但是和int a冲突
 * @param a
 */
void t(const int &a) {
    cout << "const int &a:" << a << endl;
}

    int a = 11;
    int &b = a;
//    t(10);//引用、常量引用和普通变量 不算做重载
    t(b);//int &a:11 普通引用和常量引用算作重载
    const int &d = a;
    t(d);//const int &a:11
```

1. 指针和常量指针与指针常量重载的问题

指针和int a或者int &a不会发生冲突，算重载，但是常量指针和普通指针，常量指针指向的是常量，指针指向的是变量，变量和常量是两个类型，不同的类型就算做重载。

指针常量，指针的指向地址不能改变，但是内容可以变化，那么和int * a 没有区别都是指向了变量所以它们两个会发生冲突,而且指针常量必须在初始时进行赋值

```cpp
/**
 * 指针算重载函数
 */
void t(int *a) {
    cout << "int *a:" << *a << endl;
}
/**
 * 常量指针也算重载，因为const int *a 的指向的内容不能改变是一个常量，int * a是变量，可以重载
 * @param a
 */
void t(const int *a) {
    cout << "const int *a:" << *a << endl;
}

/**
 * 指针常量 指针指向的地址不变，内容可以变，和变量 int * a 看成一致了 会冲突，int * const a 和 int * a 都是变量
 * @param a
 */
//void t( int * const a) {
//    cout << "a：" << *a << endl;
//}
```

1. 函数重载二义性的问题

如下代码，存在二义性问题，假如test(10) 不传递y的值，那么编译器不知道调用哪个函数，但是它们确实是函数重载

```cpp
void test(int x) {

}

//void test(int x, int y = 10,int d) 错误的  实参一定要在默认参数前面
void test(int x, int y = 10) {
    cout << x << y << endl;
}
```

可以通过定义函数指针来解决该问题，代码如下：

```cpp
//函数的别名 解决二义性
typedef void (*func)(int a, int b);

typedef void (*func2)(int a);

    func f1 = test;//上面加了* 这里了就不用再加了
//    func *f2 = test;
    f1(10, 11);

    func2 f2 = test;
    f2(1);
```



1. 占位参数

```cpp
/**
 * 占位参数 int = 0没有变量名 函数体是用不到的
 * 意义：可以传值也可以不传
 * 为以后程序的扩展留下线索
 * 兼容C语言中可能出现的不规则写法,兼容C
 */
void test2(int a, int b, int = 0) {

}

test2(1, 1, 1);
```



### this 特性

C++中的this和Java中的this基本一致，没有区别。在C++中this实际上是成员函数的一个形参，在调用成员函数时将对象的地址作为实参传递给this，不过this这个形参是隐式的，它并不出现在代码中，而是在编译阶段由编译器默默地将它添加到参数列表中。

this作为隐式形参，本质上是成员函数的局部变量，所以只能在成员函数的内部，并且只有在通过对象调用成员函数时才给this赋值。

在C++中类都有一个头文件.h文件

```cpp
//if not define
#ifndef CPPDEMO_THIS_H
//定义宏 CPPDEMO_THIS_H
#define CPPDEMO_THIS_H


class This {
public:
    int a;
private:
    int b;
protected:
    int c;

public:
    int getAge();

    void setAge(int a);

};

//防止多重导入 影响编译效率 其他文件导入头文件，判断宏是不是存在，如果存在 直接返回。不存在则定义宏，编译文件
#endif //CPPDEMO_THIS_H
```

由.cpp实现.h文件

```cpp
#include "include/This.h"

//实现头文件的方法
int This::getAge() {
    return a;
}

void This::setAge(int a) {
    this->a = a;//C++ this->a   java : this.a
}
```



注意：

- this是const指针，它的值不能被修改，一切视图修改指针的操作都是不允许的
- this只能在成员函数内部使用，用在其他地方没有意义
- 只有当对象创建后this才有意义，this不能在static函数中使用



如何调用`This.cpp`呢？通过引入头文件`This.h` 编译器可以自动去查找实现文件，如下代码：

```cpp
#include "include/This.h"


int main() {
    This* tt = new This;//new的实例化在堆区
    tt->setAge(1);
    cout << tt->getAge() << endl;//1
}
```

### 面向对象

在C++中创建一个类（和Java是不同的）,`规范需要有.h头文件和.cpp实现文件`

```
Student.h
#include <iostream>

//student.h 头文件只写声明不写实现
class Student{
private://下面的成员和函数都是私有
    char * name;
    int age;
public://下面的成员和函数都是公开
    void setAge(int age);
    void setName(char * name);
    int getAge() const;
    char * getName();
};
Student.cpp
//实现文件
#include "student.h"//根据头文件写实现


//实现函数:
void Student::setAge(int age) {
    this->age = age;
}

void Student::setName(char *name) {
    this->name = name;
}

int Student::getAge() const {
    return this->age;
}

char * Student::getName() {
    return this->name;
}
```

注意 创建的头文件和实现文件必须添加到`CMakeLists.txt`中

```cpp
cmake_minimum_required(VERSION 3.19)
project(base_c__)

set(CMAKE_CXX_STANDARD 14)

# 会编译EXE执行程序 注意要把头文件.h和实现文件.cpp 导入(xxx.so == xxx.cpp)
add_executable(base_c__ student.h Student.cpp T6.cpp)
```

使用创建的`Student`类

```cpp
#include "student.h"
using namespace std;

//面向对象
int main(){
    //规范写法：头文件.h .hpp  -- 实现文件 .c .cpp
    Student stu1;//在栈区开辟
    stu1.setAge(100);
    stu1.setName("张三");
    cout << "name: "<< stu1.getName() << " age:"<< stu1.getAge() << endl;

    //在真正开发的时候是通过new 在堆空间开辟
    Student *student = new Student();//new -> delete    malloc -> free
    student->setName("李思");
    student->setAge(19);
    cout << "name: "<< student->getName() << " age:"<< student->getAge() << endl;
    if (student){
        delete student;//delete 必须手动释放堆空间的student
        student = NULL;
    }
    //NDK 正规的流程：xxx.so 文件是（C/Cpp）的实现文件。 .h是声明文件
    return 0;
}
```



### 命名空间

```cpp
#include <iostream>

//声明std main函数就可以直接使用里面的成员
using namespace std;

//自定义命名空间
namespace jake{
    int age = 33;
    char * name = "jakeprim";

    void show(){
        cout << "name:" << name << " age:" << age << endl;
    }
}

namespace jake2{
    int age = 33;
    char * name = "jakeprim2";

    void show(){
        cout << "name:" << name << " age:" << age << endl;
    }

    void show1(){
        cout << "name:" << name << " age:" << age << endl;
    }
}

//命名空间的嵌套问题
namespace jake3{
    namespace jake3Inner{
        void show(){
            cout << "jake3Inner" << endl;
        }
    }
}


//声明刚刚写的命名空间
using namespace jake2;

int main(){

    cout << "测试" << endl;

    jake::show();

    //局部使用
    using namespace jake;
    //直接调用
//    show();//存在重复的函数 会报错 必须使用命名::
    jake::show();
    jake2::show();

    show1();

    using namespace jake3::jake3Inner; //声明嵌套的命名空间

    jake3::jake3Inner::show();

    return 0;
}
```



### 构造函数

C++的构造函数和Java中的构造函数区别不大，C++可以将对象声明在栈和堆中。

```cpp
class Student{
public:
    //无参构造函数
    Student(){
        cout << "create" << endl;
    }
    //带参的构造函数
    Student(int a){
        cout << "create:" << a << endl;
    }
    //析构函数 当对象被销毁时调用，在该方法中释放class中的成员及对象
    ~Student(){
        cout << "destroy" << endl;
    }
};

void test(){
    Student student;//等价于Student() 声明的对象存储在栈中，

    Student *stu = new Student();//使用new声明的对象存储在堆中，需要手动释放堆中的对象
}

int main() {
    test();//create create destroy 函数执行完毕在栈中生成的对象会自动释放，但是堆中的对象需要手动释放

    return 0;
}
```

C++中常见的构造函数几种形式如下：

```cpp
    //声明在栈区的对象
    Student student;//等价于Student() 声明的对象存储在栈中，
    Student student1(10);//括号法 有参的构造函数
    Student student2 = Student();//显示法
    Student student21 = Student(11);//显示法
    Student student3 = 10;//隐式法：等价于Student(10); 隐式调用默认构造  不推荐使用 引起歧义
	////    Student student = "a";explicit 不能隐式调用 防止产生歧义，一般开发中都不会使用隐式调用
    //explicit 不能通过隐式调用
    explicit Student(char *a) {
        cout << "create double:" << a << endl;
    }
    Student student4[4];//声明数组 有实例化对象调用了默认的构造函数  在栈区实例化
```

C++ 中会有默认的两个构造函数，一个是无参构造函数，一个是拷贝构造函数.

默认的拷贝构造函数是**浅拷贝**

```cpp
    //无参构造函数
    Student() {
        cout << "create" << endl;
    }
    //拷贝函数 常量引用 不可以改变拷贝对象的值  对参数赋值时会调用该拷贝函数，参数一定是常量引用
//    Student(Student const &student) {
//        cout << "const &student" << endl;
//        会进行浅拷贝 将值赋值给   C++ 内部会默认实现拷贝函数
//        a = student.a;
//    }
    /**
     * 也是拷贝函数 且优先级比const高，但是在程序上必须使用安全的const作为拷贝函数
     * @param student
     */
//    Student(Student &student){
//
//    }


    Student student;//create
    student.a = 10;
    test2(student);//实例化了两次  析构函数调用了两次
    Student student1(student);//会调用默认的拷贝函数
    Student student2 = student;//赋值也会调用默认的考本函数
```



拷贝构造函数的意义：在进行函数参数传递一个对象或者给一个对象赋值时需要使用到,在程序定义中不影响外部的拷贝对象是安全的，所以拷贝函数中的参数一定要是常量，不能改变外部的对象，新的对象只需要赋值即可。拷贝构造函数默认是常量引用，可以自己重新定义拷贝构造函数.

拷贝函数三种情况会被调用：

- 形参传递
- 实例化对象
- 返回参数

```cpp
void test2(Student student) {//形参 是实参拷贝 const &student 先声明Student对象  需要实例化Student对象
    cout << "a:" << student.a <<endl;//拿到
}
```

Java中的函数传递对象参数使用的是普通引用，所以在Java中可以在函数改变外部对象。C++默认使用的都是常量引用，不会影响外部函数，导致不可预期的错误。



```cpp
    Student s1 = Student(10);
    Student s2 = Student(s1);
    s2.a = 101;
    //s1 和 s2 谁先销毁呢？ s2先销毁，栈先进后出
    /**
     * create int:10
        const &student
        destroy101
        destroy10
        拷贝函数
     */
```

深拷贝实现：默认的拷贝函数是浅拷贝实现，如果需要实现深拷贝需要重写拷贝函数

字符串或者对象就需要通过深拷贝来实现

```cpp
Student(char *name, int age) {
        this->a = age;
        //由于外部传递过来的字符串存储在常量区域，name是一个常量指针，无法通过指针修改字符串的内容
        this->name = (char *) (malloc(strlen(name) + 1));//在堆区申请了一块内存
        strcpy(this->name, name);//复制到this.name中去
    }    
//默认拷贝函数 常量引用 不可以改变拷贝对象的值  对参数赋值时会调用该拷贝函数，参数一定是常量引用
    Student(Student const &student) {
        cout << "const &student" << endl;
        //会进行浅拷贝 将值赋值给   C++ 内部会默认实现拷贝函数
        this->a = student.a;
        //新对象 也会在堆申请一块内存 各指自己的
        this->name = (char *) (malloc(strlen(student.name) + 1));//在堆区申请了一块内存
        strcpy(this->name, student.name);//复制到this.name中去
    }

    //析构函数 当对象被销毁时调用，在该方法中释放class中的成员及对象
    ~Student() {
        cout << "destroy" << this->a << endl;
        //第二个对象 name不为空，释放了name的堆空间，而到了第一个对象销毁时，也会调用free这时候就会发现问题
        //必须重写拷贝函数
        if (this->name != nullptr) {
            free(this->name);
            this->name = nullptr;
        }
    }
```

字符串为什么需要重写拷贝函数呢？

```cpp
    Student s1("jake23", 1);//这里传递的字符串 是在常量区中，那么参数指针就成了常量指针了，所以需要重新分配空间存储
```



参数列表：

```cpp
    //无参构造函数 参数列表 给参数默认值
    Student() : a(10), b(20), c(30) {

    }

    //有参构造函数 参数列表
    Student(int a, int b, int c) : a(a), b(b), c(c) {

    }
```

### 析构函数

析构函数主要是在销毁对象的时候调用的，销毁内存处理等

```cpp
    //析构函数 当对象被销毁时调用，在该方法中释放class中的成员及对象
    ~Student() {
        cout << "destroy" << this->a << endl;
        //第二个对象 name不为空，释放了name的堆空间，而到了第一个对象销毁时，也会调用free这时候就会发现问题
        //必须重写拷贝函数
        if (this->name != nullptr) {
            free(this->name);
            this->name = nullptr;
        }
    }
```

new 关键字会将对象实例化到堆中，没有new的关键字会将对象实例化到栈中，堆中需要手动释放内存搭配delete关键字使用，在之前学习C的时候是通过malloc/free进行申请堆内存和释放堆内存，new/delete是一样的原理

```cpp
    Student *s = new Student;
    delete s;//释放对象 new delete / malloc free

    //数组 实例化10个对象都是在堆区
    Student *sArr = new Student[10];
    //释放
    delete[]sArr;
```

### C ++ 常见的数据结构与函数



注意数组名和指针的区别：

数组名是一个指针常量，不能改变指针的指向

```cpp
int i, *pa, a[] = {3,4,5,6,7,3,7,4,4,6};
pa = a;
for (i = 0; i <= 9; i++)
{
   printf("%d\n", *pa);
   pa++; /*注意这里，指针值被修改*/
}
可以看出，这段代码也是将数组各元素值输出。不过，你把循环体{}中的pa改成a试试。你会发现程序编译出错，不能成功。看来指针和数组名还是不同的。其实上面的指针是指针变量，而数组名只是一个指针常量。这个代码与上面的代码不同的是，指针pa在整个循环中，其值是不断递增的，即指针值被修改了。数组名是指针常量，其值是不能修改的，因此不能类似这样操作：a++。
```

#### ArrayList实现

```cpp
class ArrayList {
public:
    int d = 11;
    ArrayList();

    //explicit 不能通过隐式调用
    explicit ArrayList(int capacity);

    //拷贝函数
    ArrayList(const ArrayList &arrayList);

    //析构函数
    ~ArrayList();

    void add(int val);

    void add(int val, int index);

    int get(int index);

    int remove(int index);

    //const 定义为常函数
    int getLength() const;

    bool isEmpty() const;

    void resize();

    void toString();

private:
    int size;//容器的大小
    int realSize;//真实的数组长度
    int *arr;//这里不能使用数组，因为数组名是arr指针常量，不能对arr重新赋值， 指针是指针变量，而数组名只是一个指针常量
    mutable int c = 10;//可以在常函数中修改的变量 需要使用mutable进行修饰
};
```

函数具体实现代码如下：

```cpp
ArrayList::ArrayList(int capacity) {
    this->size = capacity;
    this->realSize = 0;
    //在堆区申请数组
    this->arr = new int[this->size];//在堆中开辟的一块空间 存储的是一个int[size] 数组，arr指向数组的首地址
}

ArrayList::ArrayList() {
    this->size = 16;
    this->realSize = 0;
    this->arr = new int[this->size];
}

ArrayList::ArrayList(const ArrayList &arrayList) {
    this->size = arrayList.size;
    this->realSize = arrayList.realSize;
    this->arr = new int[arrayList.size];
    //将数组的值赋值到arr中
    for (int i = 0; i < this->size; ++i) {
        this->arr[i] = arrayList.arr[i];//arrayList.arr[i]他也是指针  this->arr[i] 是指针
    }
}

ArrayList::~ArrayList() {
    if (this->arr != nullptr) {
        delete[] this->arr;
        this->arr = nullptr;
    }
}

void ArrayList::add(int val) {
    //添加到末尾
    add(val, this->realSize);
}

void ArrayList::add(int val, int index) {
    if (index < 0 || index > size) {
        return;
    }
    //判断容量是否够大 不够进行扩容
    if (this->realSize >= size * 0.75) {
        resize();
    }
    this->arr[index] = val;// 等价于   *((this->arr)+index) = val
    this->realSize++;//数据量大小+1
}

void ArrayList::resize() {
    cout << "----resize-----" << endl;
    int netLength = size * 2;
    int *p = new int[netLength];
    //拷贝数据
    for (int i = 0; i < size; ++i) {
        *(p + i) = this->arr[i];
    }
    //释放之前的数组
    delete[] this->arr;
    //重新赋值
    this->arr = p;
    this->size = netLength;
}

int ArrayList::get(int index) {
    if (index < 0 || index >= realSize) {
        return -1;
    }
    return this->arr[index];
}

int ArrayList::remove(int index) {
    if (index < 0 || index >= realSize) {
        return -1;
    }
    //如何移除呢？循环往前移动
    int result = this->arr[index];
    for (int i = index; i < size - 1; ++i) {
        this->arr[i] = this->arr[i + 1];
    }
    this->realSize--;
    //判断缩减容量
    return result;
}

void ArrayList::toString() {
    cout << "[ ";
    for (int i = 0; i < realSize; ++i) {
        cout << arr[i] << ", ";
    }
    cout << " ] " << endl;
}

bool ArrayList::isEmpty() const {
    return realSize == 0;
}

/**
 * 定义为常函数 不能够修改内部的变量
 * @return
 */
int ArrayList::getLength() const {
//    realSize = realSize - 1; 这样会报错 不能修改函数内部的所有变量
    c = 11;//mutable 修饰的变量可以在常函数中修改
    return realSize;
}
```

#### 常函数

通过const 修饰的函数，函数的内部任何变量都不允许修改，除了mutable修饰的变量

```cpp
mutable int c = 10;//可以在常函数中修改的变量 需要使用mutable进行修饰
/**
 * 定义为常函数 不能够修改内部的变量
 * @return
 */
int ArrayList::getLength() const {
//    realSize = realSize - 1; 这样会报错 不能修改函数内部的所有变量
    c = 11;//mutable 修饰的变量可以在常函数中修改
    return realSize;
}
```

#### 常对象

通过const修饰的对象叫做常对象，在常对象中对任何变量的修改都是违法的，常对象只能调用常函数，因为常函数不允许修改变量，注意通过mutable修饰的变量还是可以修改的(mutable C++添加了更加灵活的特性)

```cpp
    //常对象 对任何变量的修改都是违法的 一般用在不能改变的对象上，比如单例模式的一些配置信息等
    const ArrayList arrayList1 = ArrayList();
//    arrayList1.d = 11; //错误的
//    常对象只能调用常函数，其他函数无法调用.
    arrayList1.getLength();
```

#### 单例模式

C++ 实现单例模式

```cpp
/**
 * 单例模式实现
 */
class Instance {
public:
    static Instance *getInstance() {
        return instance;
    }

private:
    static Instance *instance;
private:
    //保护构造方法
    Instance() {}

    //保护构造拷贝函数 不让外界去声明 这里和java是不一样的
    Instance(const Instance &instance) {

    }
};

//初始化静态属性 在类外初始化
Instance *Instance::instance = new Instance;
```

#### 友元函数

在定义一个类的时候，可以把一些函数（包括全局函数和其他类的成员函数）声明为“友元”，这样那些函数就成为该类的友元函数，在友元函数内部就可以访问该类对象的私有成员了。

将全局函数声明为友元的写法如下：

- `friend  返回值类型  函数名(参数表);`

将其他类的成员函数声明为友元的写法如下：

- `friend  返回值类型  其他类的类名::成员函数名(参数表);`

但是，不能把其他类的私有成员函数声明为友元。

```cpp
class Car;//提前声明Car
class Driver {
private:
    void pfun();    
public:
    void ModifyCar(Car *car);
    void Not(Car *car);
};

class Car {
public:
    int a;
private:
    int price;

    friend int Most(Car cars[], int total);//声明友元函数
    friend void Driver::ModifyCar(Car *car);//声明友元函数
    //    friend void Driver::pfun();//其他类的私有成员不能设置为友元函数
};

/**
 * 类定义的友元函数可以访问私有变量
 * @param car 
 */
void Driver::ModifyCar(Car *car) {
    car->price += 1000;//ModifyCar作为友元函数 可以访问car的私有变量
}

/**
 * 外部定义的友元函数可以访问私有变量
 * @param cars 
 * @param total 
 * @return 
 */
int Most(Car cars[], int total)  //求最贵气车的价格
{
    int tmpMax = -1;
    for (int i = 0; i<total; ++i)
        if (cars[i].price > tmpMax)
            tmpMax = cars[i].price;
    return tmpMax;
}

void Driver::Not(Car *car) {
    //无法访问私有变量
    car->a;
}
```

#### 友元类

友元类的定义非常简单如下代码：

```cpp
class CCar{
private:
    int price;
    friend class CDriver;//定义友元类 可以访问CCar的私有成员
};

class CDriver{
public:
    CCar mCar;
    void Most(){
        mCar.price+=100;
    }

private:
    void Host(){
        mCar.price+=1;
    }
};
```

### 运算符重载

#### 算术运算符重载

运算符重载需要使用关键字`operator[...]`

```cpp
/**
 * 算数运算符重载
 */
class complex {
public:
    complex();

    complex(double real, double imag);

private:
    double m_real;
    double m_imag;
public:
    //声明运算符重载 参数是一个常量引用，重载函数是常函数，不能修改函数内部的所有变量
    complex operator+(const complex &A) const;

    //打印函数也是一个常函数，不能修改变量
    void display() const;
};

complex::complex() : m_real(0), m_imag(0) {

}

complex::complex(double real, double imag) : m_real(real), m_imag(imag) {}

complex complex::operator+(const complex &A) const {
    complex B;
    B.m_real = this->m_real + A.m_real;
    B.m_imag = this->m_imag + A.m_imag;
    return B;
}

void complex::display() const {
    cout << m_imag << " + " << endl;
}
```

使用方式如下：

```cpp
    complex c1(4.3, 5.9);
    complex c2(2.4, 3.7);
    complex c3;
    c3 = c1 + c2;//+操作会执行operator+()
    c3.display();//9.6 + 6.7
```

#### 指针运算符重载

```cpp
class Person{
public:
    Person(string name){
        this->name = name;
    }

    ~Person(){

    }

    const string & get_name(){
        return name;
    }

private:
    string name;
};

//模板
template<class classtype>
class SmartPoint{
private:
    classtype* pointer;
public:
    SmartPoint(classtype* pointer){
        this->pointer = pointer;
    }
    ~SmartPoint(){
        if (this->pointer != NULL){
            delete this->pointer;
        }
    }

    //重载运算符 ->
    classtype* operator->(){
        return this->pointer;
    }

    //重载解引用
    classtype& operator*(){
        return *(this->pointer);
    }
};
```

使用方式如下：

```cpp
	SmartPoint<Person> p = SmartPoint<Person>(new Person("lili"));
    cout << p->get_name() << endl;
    cout << (*p).get_name() << endl;
```

#### new运算符重载

```cpp
/**
 * new运算符重载
 */
class CTest1{
    char m_buff[4096];
};

//重载new运算符，将对象创建到全局的buffer中去
char buf[4100];//将创建的对象存储在buf开辟的内存中

class CTest2{
    char m_buff[4096];
public:
    void *operator new(size_t size){
        return (void *)buf;
    }
    //delete 重载
    void operator delete (void *p){

    }
};
```

性能对比如下：

```cpp
 //测试CTest1的性能
    clock_t count = clock();
    for (int i = 0; i < 0x5ffff; ++i) {
        CTest1 *c = new CTest1();
        delete c;
    }
    cout << "Interval = " << clock() - count << endl;//Interval = 52

    clock_t count1 = clock();
    for (int i = 0; i < 0x5ffff; ++i) {
        CTest2 *c = new CTest2();
        delete c;
    }
    cout << "Interval = " << clock() - count1 << endl;//Interval = 26 性能会更好
```



#### 赋值运算符重载



```cpp
class Customer {
private:
    int id;
    char *name;
public:
    Customer() : id(0), name(NULL) {}

    Customer(int _id, char* _name) {
        id = _id;
        //注意这里不能直接赋值的，需要重新申请一块内存给name，并且将值复制给name
        name = new char[strlen(_name) + 1];
        strcpy(name, _name);
    }

    //拷贝函数
    Customer(const Customer &str) {
        cout << "copy constructor" << endl;
        id = str.id;
        name = new char[strlen(str.name) + 1];
        strcpy(name, str.name);
    }

    //重载赋值运算符
    Customer &operator=(const Customer &str) {
        cout << "operator:" << this << " str:" << &str << endl;
        if (this != &str) {
            if (name != NULL) {
                delete name;
            }
            this->id = str.id;
            name = new char[strlen(str.name) + 1];
            strcpy(name, str.name);
        }
        return *this;
    }
    ~Customer(){
        delete[] name;
    }
};
```

如下使用方式：

注意如果在声明变量的时候直接赋值=，不会调用重载的赋值运算符

```cpp
	char * name = "lll";
    Customer cc1(1, name);
    Customer cc2;
    cc2 = cc1;//operator:0x63fc60 str:0x63fc70   这里才会调用重载的赋值运算符
    Customer cc3 = cc2;//copy constructor  注意直接给变量赋值，调用了拷贝函数
```

### 智能指针

关于智能指针后续会专门讲解，智能指针属于STL库中的后续分析STL库的源代码

```cpp
//shared_ptr 智能指针采用引用计数的方式，避免指针缺陷，内存泄漏、野指针等问题
    std::shared_ptr<Person> p1(new Person("meimei"));//Person("meimei")的引用计数为1
    std::shared_ptr<Person> p2 = std::make_shared<Person>("lilei");
    p1.reset(new Person("guoguo"));
    std::shared_ptr<Person> p3 = p1;//shared_ptr 重载了-> 运算符
    cout << "p3 is" << p3->get_name();//-> 获取的是p1.get_name();
```