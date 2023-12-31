# C | 深入理解指针

指针：存储的内存地址，只能是地址，指向了内存的某一个区域

CPU 访问内存时需要的是地址，而不是变量名和函数名！变量名和函数名只是地址的一种助记符，当源文件被编译和链接成可执行程序后，它们都会被替换成地址。编译和链接过程的一项重要任务就是找到这些名称所对应的地址。

数据在内存中的地址也称为[指针](http://c.biancheng.net/c/80/)，如果一个变量存储了一份数据的指针，我们就称它为**指针变量**。

在C语言中，允许用一个变量来存放指针，这种变量称为指针变量。指针变量的值就是某份数据的地址，这样的一份数据可以是数组、字符串、函数，也可以是另外的一个普通变量或指针变量。

### 理解单个指针变量:

单个指针变量p，存储变量a的内存地址,`*p`获取存储的内存地址的值也就是 `10`

```c
    int a = 10;//实体变量
    p = &a;//指针p赋值为a的内存地址
    //查看a的地址
    printf("&a = %p\n", &a);//&a = 000000000061FDE4
    //查看指针变量p存储的内容 就是变量a的内存地址
    printf("p=%p\n", p);//p=000000000061FDE4
    //指针变量p也是有地址的因为变量也会存在内存的某个区域,*p取出改地址的实体也就是存储的值
    printf("p=%p,*p=%d\n", &p, *p);//p=000000000061FDE8,*p=10
```

假设改变了*p = 100 那么变量a的值会变吗？答案是：会  因为*p 改变的是a内存地址的值，那么a的值也会变

```c
    //改了*p的值，原来的值也会改变，因为都是同一个地址.相当于改变了内存地址000000000061FDE4 存储的值
    *p = 100;
    printf("a=%d,*p=%d\n", a, *p);//a=100,*p=100
```

### 指针的类型

指针是有类型的，但是**指针的大小是固定的**(基础类型的大小是不一样的)，**指针类型在前，指针指向的类型在后**。

`int *p;` // 指针类型 -> int *，指向类型 -> int

`int **p;`//指针类型 -> int **,指向类型 -> int *,如下图所示：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631175803040-312a753a-4052-4799-b84e-7314a80e5781.png)



```c
 	printf("sizeof p = %llu\n", sizeof p);//int *p 查看指针的大小sizeof p = 8

    char b = 'c';
    char *c = &b;
    char d = *c;//取出指针变量c，存储地址的实际值
    printf("sizeof c = %llu\n", sizeof c);//char *p sizeof c = 8
    printf("d=%c\n", d);//d=c
```

不管是`int *` 还是` char *` 指针的大小都是固定的，与类型无关

**指针占用的字节就是固定的8个字节(64为8个字节，32位4个字节)**，**指针的大小是不变的和指针的类型无关，地址是没有长短之分的 地址是固定的长度,指针存储的就是****地址****所以大小肯定是固定的。**

注意：指针有类型，不能int类型指针存储char类型

```c
    //注意：指针是有类型的，不能char类型的指针，存储int类型的指针
    //变量a是int类型
    char *e = &a;//Incompatible pointer types initializing 'char *' with an expression of type 'int *'
```

1. 指针的步长

指针的步长：由指针的类型决定,int 类型步长为4个字节，char 类型步长为1个字节

```c
    int aa = 0xaabbccdd;
    int *p1 = &aa;
    char *p2 = &aa;
    printf("p1=%x\n",*p1);//p1=aabbccdd
    printf("p2=%x\n",*p2);//p2=ffffffdd *p2 找到aa的地址存储的值，并且当作char类型去使用

    //p1和p2都是同一个地址
    printf("p1=%p\n",p1);//p1=000000000061FDBC

    printf("p2=%p\n",p2);//p2=000000000061FDBC

    printf("p1=%p\n",p1+1);//p1=000000000061FDC0   加4个字节 int类型占4个字节

    printf("p2=%p\n",p2+1);//p2=000000000061FDBD   加1个字节 char类型占1个字节
```



如下代码：

`int **p`,注意***p 指向的是一个指针类型的变量**，而指针类型的大小永远是8个字节(64位)，而`**p` 则是具体的基础变量的值了，大小有基础类型决定。

```c
	int p = 10;
	int* p1 = &p;
	int** p4 = &p1;
	printf("大小 :%d,大小2：%d\n",sizeof(*p4),sizeof(**p4));//大小 :8,大小2：4
```



### 指针数组与数组指针

在初学习C语言的时候，经常会分不清**指针数组**和**数组指针，**他们两个有什么区别呢？我来带你彻底搞清楚

在搞清楚是指针数组还是数组指针首先要看**优先级 : ()  > [] > \***

如下代码，p2是一个指针数组,怎么看的呢？

- [] 的优先级大于 *
- 那么p2[4] 先声明了有4个空间的**数组**
- int * 就表明了数组存储的是**int类型的指针**
- **数组中存储指针那么一定是指针数组**

```c
	int* p2[4];//指针数组   p2[4] 是个数组，  int * 数组存储的是int类型的指针
```

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631175816829-9a9c6e8f-2771-4d7d-9bd5-fc5e6bef5e1b.png)

再来看如下代码：

- 括号的优先级大先和括号结合  *p3 指针

- [4] 指向了后面的数组
- 数组的类型为int

这就表示了：一个指针指向了一个int类型的数组，那么他就是数组类型的指针就是**数组指针**

```c
int(*p3)[4];//数组指针  
```

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631175825915-53137f05-5d82-4379-8863-713c2ad6e253.png)

指针数组的使用如下：

```c
int main() {
    //指针数组 数组存放的是指针类型
    int a[] = {10,11};
    int b[] = {20};
    int c[] = {30};

    //a = &a[0] 正好是int类型的指针
    int * d[] = {a, b, c};//指针数组 存放的是a的地址 *d == &a  **d = *(&a)  &a = &a[0]  *(&a[0])
    printf("&a[0]=%p,&d[0]=%p,d[0]=%d,*(&a[0])=%d\n",&a,*d,**d,*(&a[0]));
    //&a=000000000061FE1C,&d[0]=000000000061FE1C,d[0]=10,*(&a[0])=10

    //*d+1  *d = &a[0]  *d+1 = &a[0]+1
    printf("&a[1]=%p,&d[0]=%p,a[1]=%d\n",&a+1,*d+1,*(*d+1));
    //&a[1]=000000000061FE20,&d[0]=000000000061FE1C,a[1]=11

    //如果要取a[0] 不能写 *(&a) 要改为 *(&a[0])
    printf("%d,&a[0]=%p,&a=%p\n",*(&a[0]+1),&a[0],&a);
    //11 &a[0]=000000000061FE18,&a=000000000061FE18

    //推断 *d[0] = **d  *d[1] = **(d+1)   *d = &a
    printf("d[0]=%d,%d,d[1]=%d,%d\n",*d[0],**d,**(d+1),*d[1]);
    //d[1]=20,20

    //**(d+1) == *(*(d+1))
    printf("%d\n",*(*(d+1)));

    return 0;
}
```



### 野指针、空指针、悬空指针

注意野指针、空指针和悬空指针的区别

- 野指针

野指针：未初始化的指针，其指针内容为一个垃圾数

```c
	int* p7;//没有初始化没有指向任何地址，那么他就是一个野指针
	//printf("p:0x%x\n",p7);
```

- 空指针

NULL == 0,空指针不指向任何实例的对象或者函数。反过来说任何对象或者函数的地址都不可能是空指针

NULL 指针分配的区域从0x00000000 - 0x0000FFFF 这段空间是空闲的没有相应的物理存储器与之对应，对于这段空间来说，任何读写操作都是引起异常的。空指针是程序**无论在何时都没有物理存储器与之对应的地址**。

```c
int* p8 = NULL;//指针未初始化并且赋值为NULL则为 空指针
```



- 悬空指针

悬空指针：是指指针正常初始化，层指向过一个正常的对象，但是对象销毁了，该指针未置空，就成了悬空指针。

```c
	int* p8 = NULL;//指针未初始化并且赋值为NULL则为 空指针

	{
		int i = 6;
		p8 = &i;
		*p8 = 7;
		printf("----%p,%p\n",&i,p8);//----0000004A9D9DF914,0000004A9D9DF914
	}
	//需要注意的是当代码块运行完毕后 变量i就会被销毁，但是它的内存并没有清零还是存在的所以*p8还可以得到7
	//此时p8已经是一个悬空指针了，因为指向的内存地址没有任何变量持有，变量虽然被销毁了，但是该内存地址存储的信息没有清零
	int b = *p8;
	printf("b=%d\n",b);//b=7
```

### 指针表达式



理解常见的指针表达式以及运算的结果

```groovy
	/*
	指针表达式
	* ch
	* &ch
	* cp
	* &cp
	* *cp
	* *cp+1
	* *(cp+1)
	* ++cp
	* cp++
	* *++cp
	* *cp++
	* ++*cp
	* (*cp)++
	* ++*++cp
	* ++*cp++
	*/
	char ch = 'a';
	char name = ch;//ch作为右值 会取出存储的值

	char* cp = &ch;//cp 作为左值会存放指向的内存地址,而cp也有自己的地址，只不过它的值是存储的也是地址。

	char* cp1 = cp;//cp作为右值,会取存储的值：&ch

	printf("cp:%p,cp1:%p\n", cp, cp1);//cp:00000067C21FFA14,cp1:00000067C21FFA14

	printf("&cp:%p，&ch:%p\n",&cp,&ch);//cp有自己的地址 和&ch是不一样的 &cp:00000067C21FFA38，&ch:00000067C21FFA14
	*cp = 'b';//ch的值也会发生改变
	printf("%c,%c\n",ch,*cp1);//b,b

	char** cp2 = &cp;
	printf("&ch:%p,ch:%c\n",*cp2,**cp2);//&ch:000000E9984FFD24,ch:b

	char name1 = *cp;//解引用
	printf("name1:%c\n", name1);//name1:b

	//*cp+1 ==> * > + (*会先于+)
	char name2 = *cp + 1;//b (*cp) + 1 == a + 1 == b

	//*(cp+1)  ==> cp 取出 &ch + 1 地址+1，然后对这个地址解引用， 而这个指向的地址,是个野指针 拿到的是个随机值
	printf("%c\n", *(cp + 1));//随机值

	printf("%p,%p\n", ++cp, cp++);//00000067C21FFA16,00000067C21FFA14
```

### 内存管理

C 语言在内存管理上有着决定权，它不想java语言一样自动管理内存，C语言需要手动申请内存和释放内存

在计算机中内存一般分为：栈区和堆区(其实只有java语言会有一个方法区)，在C中一般定义的变量都是在栈区中，如果要定义在堆区就需要向堆区申请内存空间.

```c
	//堆区申请内存
	int* arr = malloc(16);
```

上述代码中，指针`arr`申请了16个字节的内存空间，指针的类型是int类型，内存模型如下图所示：

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631175865970-f2542bbe-6dd6-4524-9eba-7a658dceaa34.png)

```c
[[include]] <stdio.h>
[[include]] <malloc.h>
[[include]] <memory.h>

int main() {
	//堆区申请内存
	int *arr = malloc(16);
	//设置值
	memset(arr,1,16);
	//i:0 value:0,
	//i:1 value : 0,
	//i : 2 value : 0,
	//i : 3 value : 0,
	for (int i = 0; i < 4; i++) {
		printf("i:%d value:%d,\n",i,*(arr+i));
	}

	free(arr);//此时arr成了悬空指针了 必须设置为NULL
	arr = NULL;

	//malloc(size): 在内存的动态存储区域中分配一块为size字节的连续区域，返回首地址

	//calloc(n,size) 在内存的动态存储区中分配n块长度为size字节的连续区域，并且进行清零
	int* a = calloc(2, 8);//2*8也是16个字节的内存空间
	/*
	i:0 value:0 p:000001E669A6B9E0
	i:1 value:0 p:000001E669A6B9E4
	i:2 value:0 p:000001E669A6B9E8
	i:3 value:0 p:000001E669A6B9EC
	*/
	for (int i = 0; i < 4; i++) {
		printf("i:%d value:%d p:%p\n", i, *(a + i),(a+i));
	}

	//释放掉申请的内存
	free(a);
	a = NULL;

	return 0;
}
```

如下代码，实现一个数据交换的函数：这样能交换成功吗？其实是不可以的，依据内存可以思考一下。

没有交换成功，为什么呢？因为传递是值，而x和y存在swap栈帧中，而且swap执行完毕后 x 和 y都会销毁掉

```c
int swap(int x, int y) {
	printf("swap before x:%d,y:%d\n",x,y);
	int tmp = x;
	x = y;
	y = tmp;
	printf("swap after x:%d,y:%d\n",x,y);
}

int main() {
	int x = 2;
	int y = 4;
	swap(x, y);//进行x 和 y的交换
	//但是这样能交换成功吗？ 答案是 不能
	/*
	swap before x:2,y:4
	swap after x:4,y:2
	swap main:x:2,y:4
	*/
	printf("swap main:x:%d,y:%d\n",x,y);
}
```

如下图所示：main中的x和y 与 swap中的x 和 y不是同一个内存地址，只是进行了值传递，所以不能够使main中的x 和 y进行值的交换。

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631174561496-6a042f67-3f29-4037-9d7b-2062800bf513.png)



那么怎么实现两个变量的值交换呢？这时候就需要用到指针，因为指针是指向了内存地址，通过上述了解使用同一个内存地址实现值的交换。

```c
/*
传递的是内存地址
*/
int swap2(int *x, int *y) {
	printf("swap before x:%d,y:%d\n", *x, *y);
	int tmp = *x;
	*x = *y;
	*y = tmp;
	printf("swap after x:%d,y:%d\n", *x, *y);
}
```

实际的执行方式如下图所示：



![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631174724856-d6c93e79-1b56-49bf-b89d-334ad3c5b267.png)

通过传递地址，来进行值的交换

```c
	/*
	swap before x:2,y:4
	swap after x:4,y:2
	swap2 main:x:4,y:2
	*/
	swap2(&x, &y);
```

那么再来思考一下，普通变量值实现了交换，那么指针是否也可以实现交换呢？来看下面这段代码

```c
	int* a = &x;
	int* b = &y;
	printf("swap2 before  ---- main:a:%p,b:%p\n", a, b);
	//实现指针之间的交换
	swap2(a, b);//如果传递是a 和 b x和y实现了交换但是a和b的地址并没有实现交换
	printf("swap2 ---- main:x:%d,y:%d\n", x, y);
	printf("swap2 after ---- main:a:%p,b:%p\n", a, b);

/*
传递的是内存地址，数据的交换一定要传递地址
*/
int swap2(int *x, int *y) {
	printf("swap before x:%d,y:%d\n", *x, *y);
	int tmp = *x;
	*x = *y;
	*y = tmp;
	printf("swap after x:%d,y:%d\n", *x, *y);
}
```

结果如下：a 和 b并没有实现值的交换(注意这里的值：就是指针存储的地址)，但是反而指向的x和y 交换了。

```c
	/*
	swap2 before  ---- main:a:00000060BC5FF634,b:00000060BC5FF654
	swap before x:2,y:4
	swap after x:4,y:2
	swap2 ---- main:x:4,y:2
	swap2 after ---- main:a:00000060BC5FF634,b:00000060BC5FF654
	*/
```

仔细思考一下这是为什么呢？首先需要整理下思路，来看整体的内存模型

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631174956030-ad932be1-c0c8-46a0-9d04-684ec6a3488b.png)

在swap中也有两个指针分别是a 和 b，它和main中的两个指针 a 和 b完全是不同的，假设给swap传递的(a,b)其实传递的是x和y的内存地址，所以x 和 y进行了交换。

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631175061847-f380535c-e64b-43ee-864e-5d9d7974a110.png)

那么如何交换a 和 b的存储的地址呢？其实需要传递a和b他们本身的地址(&a &b),这样就实现了指针的交换

一定要记住：两个值的交换，必须要传递地址，不管是普通变量还是指针变量(指针只不过是特殊的普通变量而已)

```c
swap2(&a, &b);
	/*
	swap2 before  ---- main:a:00000057EB0FF634,b:00000057EB0FF654
	swap before x:-351275468,y:-351275436
	swap after x:-351275436,y:-351275468
	swap2 ---- main:x:2,y:4
	swap2 after ---- main:a:00000057EB0FF654,b:00000057EB0FF634
	*/
```

![img](https://cdn.nlark.com/yuque/0/2021/png/375694/1631175137831-6034bbd4-5ea9-4698-a88d-6d42b2aabc62.png)