# C 语言的缺陷到C++的改进

### C 语言的字符语法陷阱

> C 语言中字符语法陷阱，格外注意单引号和双引号的问题。C语言是高级语言中的低级语言，优点 小巧、高效、接近底层；缺点：细节和陷阱比较多

代码如下所示：

- c1 字符类型：注意到c1赋值了'yes' 三个字符，但是编译器认为是正确的，单引号 '' 一个字符， “y”等价于 ('y','\0') 既然是一个字符 为什么能放yes?，这里会导致**截断结果**（不同的编译器的结果不同）: s 保留最后一个字符

- c2 是字符类型，双引号是字符串 不能直接赋值，而是需要通过指针进行赋值，slash就是正确的写法。
- slash2 字符类型的指针，注意是指针，**指针的赋值必须是内存地址**，而不能将字符直接赋值给指针，正确的做法slash3这样处理。

```c
	char c1 = 'yes';//正确  
	//char c2 = "yes"; //错误的 "" 表示是个字符串 赋值给字符char变量 
	const char* slash = "/";//正确 ==> 指向了一块内存: （'/','\0'）

	//char 类型的不能初始化 const char* 类型的变量,字符类型 给到了一个指针
	//const char* slash2 = '/';//错误的 可以这样做
	const char* slash3 = &c1; //将char的地址 赋值给指针

	cout << c1 << endl;//s
	cout << *slash3 << endl;//s
```

> C++兼容C语言，同时推出即高效又易于大规模开发的机制，string类的使用 规避C的陷阱

```cpp
string s(1, 'yes');//定义给字符 只包含1个
string s1(3,'yes');// 输出三个s
cout << s << " | " << s1 << endl;// s | sss
string s2(1, 'y');
cout << s2 << endl;// y
string s3("/");
cout << s3 << endl;// /
string s4(1,'/');
cout << s4 << endl;// /
string s5("yes");
cout << s5 << endl;// yes
```

### C 语言数组陷阱

> 数组在作为参数时的退化行为，数组在作为参数时会退化为指针，C语言这样做的原因时在空间上进行节省，当将数组传递给函数时，不需要将整个容器做数据转移，而是直接退化为指针指向数组的首地址。

如下代码计算数组的总和：在C语言中，如果要拿到数组的长度就需要通过`sizeof()`进行计算，没有直接获取数组size的方法

```c
int array[] = {10,20,30,40,50,60,70,80,90,100};
int len = sizeof(array)/sizeof(array[0]);//数组的长度 = 10
cout << average1(array) << endl;//15
cout << average2(array,len) << endl;//55
```

计算总和函数：注意如下函数就是C程序的一个缺陷，C语言会自动将数组退化为一个指针，这时候在函数内部获取数组的长度就会出现错误

```c
double average1(int array[10]) {//int array[10] 参数会退化为指针不是一个数组，指针的大小固定为4(x86) 8(x64),
	//指针的指向数组的首地址 -> 10 int类型，所以 sizeof = 4，这里一定要注册C程序的缺陷
	double result = 0;
	int len = sizeof(array) / sizeof(array[0]);
	cout << "average1 In avaerage :" << len << endl; //average1 In avaerage :2
	for (int i = 0; i < len; i++) {
		result += array[i];
	}
	return result/len;//
}
```

正确的做法：将数组的长度通过参数进行传递

```c
double average2(int array[10],int len) {//通过外部 传递数组长度,array同样会退化为指针
	double result = 0;
	//int len = sizeof(array) / sizeof(int);
	cout << "average2 In avaerage :" << len << endl;//average2 In avaerage :10
	for (int i = 0; i < len; i++) {
		result += array[i];
	}
	return result / len;
}
```

C++是如何改进这个问题呢？

> C++的解决方案：STL容器及引用，实现底层包装，保证效率的同时，保证简单和安全

```cpp
[[include]] <vector>
vector<int> vt{10,20,30,40,50,60,70,80,90,100};
cout << average3(vt) << endl;
```

计算总和函数实现如下：

```cpp
//使用vector高阶容器 可以立刻获取容器的尺寸，参数时引用
double average3(vector<int> &v) {
	double result = 0;
	vector<int>::iterator it = v.begin();
	//auto it = v.begin();
	for (; it != v.end(); ++it) {
		result += *it;
	}
	return result / v.size();
}
```

vector 底层包装了特别多的实用的方法，还可以通过for-i循环来实现

```cpp
double average4(vector<int>& v) {
	double result = 0;
	for (unsigned int i = 0; i < v.size(); i++)
	{
		result += v[i];
	}
	return result / v.size();
}
```

使用vector实现二维数组的求和：

```cpp
vector<vector<int>> vv2D{8,vector<int>(12,3)};//8个维度的vector,每个vector有12个3
double average42DV(vector<vector<int>> &vv) {
	double result = 0;
	unsigned int size = 0;
	for (unsigned int i = 0; i < vv.size(); ++i)
	{
		for (unsigned int j = 0; j < vv[i].size(); ++j) {
			result += vv[i][j];
			size += 1;
			cout << vv[i][j] << " ";
		}
		cout << endl;
	}
	return result / size;
}
```

### C 语言移位运算的问题

> C在设计移位操作时需要考虑整数表示的上下文环境, 在进行移位运算存在逻辑右移和算术右移的问题

如下代码所示：右移操作的补位是根据右侧是1还是0进行补位，这样就和预期的效果不一致，C语言这里并没有进行严格的限制，可能不同的编译器处理不同，为了保证和预期的效果一直，C语言这里右移操作使用无符号数

```c
	char a1 = 0x63;//16进制 0x63 ->  0110 0011
	a1 = (a1 << 4);//左移4位
	printf("0x%x\n", a1);//0x30 ->  0011 0000
	//右移4位
	a1 = 0x63;
	a1 = (a1 >> 4);
	printf("0x%x\n", a1);//0x6 -> 0000 0110 (按照右侧第一位是0 进行补位 所以前面是0补位) 算术右移

	char a2 = 0x95;//16进制 0x95 ->  1001 0101
	a2 = (a2 << 4);//左移4位
	printf("0x%x\n", a2);//0x50 ->  0101 0000
	//右移4位
	a2 = 0x95;
	a2 = (a2 >> 4);
	printf("0x%x", a2);//得到的结果和预期的不一致 0xfffffff9 -> 1111 1001 (按照右侧的第一位是1 进行补位 所以前面是1补位) 算术右移
```

如何下代码所示：

```c
	//那么如何保证都是逻辑右移呢? 通过unsigned 无符号数的设置
	unsigned char a3 = 0x63;//16进制 0x63 ->  0110 0011
	a3 = (a3 << 4);//左移4位
	printf("0x%x\n", a3);//0x30 ->  0011 0000
	//右移4位
	a3 = 0x63;
	a3 = (a3 >> 4);
	printf("0x%x\n", a3);

	unsigned char a4 = 0x95;
	a4 = (a4 << 4);//左移4位
	printf("0x%x\n", a4);
	//右移4位
	a4 = 0x95;
	a4 = (a4 >> 4);
	printf("0x%x\n", a4);//0x9 -> 0000 1001 
```

> 进行移位操作时需要注意：移位操作位数的限制-移位数大于0，小于位数

如下代码：char最大只有8位的内存空间，如果1<<8就会超出最大的位数，会导致数据清0，最终得到的结果就是0x0,所以在C语言进行移位操作时，一定要注意移位数小于位数

```c
//权限设置：0000 0000  移位操作位数的限制：移位数大于0，小于位数；
const unsigned char priv = 0xFF;//默认是所有权限
const unsigned char P_BAKUP = (1<<6);
const unsigned char P_ADMIN = (1<<7);//(1<<8)超出了char的最大的内存空间只有8位 数据会清0
if (priv & P_BAKUP)
{
	cout << "BAKUP" << endl;
}
if (priv & P_ADMIN)
{
	cout << "ADMIN" << endl;
}
```

上述C语言在移位操作存在的一些缺陷，稍不注意就会出现问题。

C++中的解决方案使用`bitset`来解决移位操作的问题，bitset打印出来就是直接的二进制数，通过bitset<n> n可以设置想要的位数，不会存在C语言中的右移问题和移位数超出了位数等缺陷。

```cpp
	//权限设置：0000 0000  移位操作位数的限制：移位数大于0，小于位数；
	bitset<8> priv = 0xFF;
	bitset<8> P_BAKUP = (1 << 6);
	bitset<8> P_ADMIN = (1 << 7);//(1<<8)超出了char的最大的内存空间只有8位 数据会清0
	//输出 直接以2进制的形式展示
	cout << priv << endl;//11111111
	cout << P_BAKUP << endl;//01000000
	cout << P_ADMIN << endl;//10000000

	if ((priv & P_BAKUP) == P_BAKUP)
	{
		cout << "BAKUP" << endl;
	}
	if ((priv & P_ADMIN) == P_ADMIN)
	{
		cout << "ADMIN" << endl;
	}
```

### C语言类型转换问题

