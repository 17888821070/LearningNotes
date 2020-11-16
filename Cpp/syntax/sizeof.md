# sizeof

## 定义

`sizeof` 是一个操作符，返回一个对象或类型所占的内存字节数

## 基础数据类型

`short、int、long、float、double` 的内存大小是和系统相关

## 结构体

结构体在内存组织上是按顺序排列的

结构体的 `sizeof` 涉及到字节对齐问题，字节对齐有利于加快计算机的取值速度

字节对齐：
- 结构体变量的首地址能够被其最宽基本类型成员的大小所整除

- 结构体的每个成员相对于结构体首地址的偏移量（offset）都是成员大小的整数倍，如有需要，编译器会在成员之间加上填充字节

- 结构体的总大小为结构体最宽基本类型成员大小的整数倍，如有需要，编译器会在最末一个成员后加上填充字节

- 类中有虚函数，则添加虚指针，排在类的内存布局的最开始

- 继承时，基类的成员排在基类内存布局的最开始

空结构体（不含数据成员）的 `sizeof` 值为 1

```cpp

struct S1
{
    char a;
    int b;
};
sizeof(S1); // 值为8，字节对齐，在char之后会填充3个字节
 
struct S2
{
    int b;
    char a;
};
sizeof(S2); // 值为8，字节对齐，在char之后会填充3个字节
 
struct S3
{
    int b;
    char a;
    char b;
};
sizeof(s3); // 值为8，字节对齐，在char之后会填充2个字节

struct S4
{
};
sizeof(S4); // 值为1，空结构体也占内存
```

### 联合体

联合体在内存组织上是重叠式，各成员共享一段内存，整个联合体的sizeof也就是每个成员sizeof的最大值

```cpp
union u
{
    int a;
    float b;
    double c;
    char d;
};
 
sizeof(u); // 值为8
```

## 数组

数组的 `sizeof` 值等于数组所占用的内存字节数

- 当字符数组表示字符串时，其 `sizeof` 值将 `/0` 计算进去

- 当数组为形参时，其 `sizeof` 值相当于指针的 `sizeof` 值

```cpp
char a[10];  // sizeof(a) == 10
char n[] = "abc";  // sizeof(n) == 4

void funcN(char b[])
{
    int cN = sizeof(b); //cN = 4，理由同上。
}
```

## 普通指针

指针的内存大小当然就等于计算机内部地址总线的宽度

```cpp
char *b = "helloworld";
char *c[10];
double *d;
int **e;
void (*pf)();  

cout<<sizeof(b)<<endl;//指针指向字符串，值为4
cout<<sizeof(*b)<<endl; //指针指向字符，值为1
cout<<sizeof(d)<<endl;//指针，值为4
cout<<sizeof(*d)<<endl;//指针指向浮点数，值为8
cout<<sizeof(e)<<endl;//指针指向指针，值为4
cout<<sizeof(c)<<endl;//指针数组，值为40
cout<<sizeof(pf)<<endl;//函数指针，值为4
```

## 类的指针和引用

指针的解应用和引用都是数据类型大小，而不是指向所以大小

```cpp
class A {
public:
	virtual void h() {}
};

class  B: public A
{
public:
	int i;
	virtual void h() {}

};

B b;
A& a = b;
B& bb = b;
A* ap = &b;
B* bp = &b;
cout << sizeof(a) << endl;    // 4
cout << sizeof(bb) << endl;   // 8
cout << sizeof(*ap) << endl;  // 4
cout << sizeof(*bp) << endl;  // 8
```

## 函数

`sizeof` 也可对一个函数调用求值，其结果是函数返回值类型的大小，需要传入实参但函数并不会被调用，但不可以对返回值类型为空的函数求值

## 类

- 空类的大小为 1 字节

```cpp
class A{};
cout<<sizeof(A)<<endl;  // 1
```

- 一个类中，虚函数本身、成员函数（静态和非静态）和静态数据都不占用类对象的存储空间

```cpp
class A
{
public:
    char b;
    virtual void fun() {};
    static int c;
    static int d;
    static int f;
};
cout<<sizeof(A)<<endl;  // 内存对齐 8 + 8 = 16
```

- 对于包含虚函数的类，不管有多少个虚函数，只有一个虚指针

- 普通继承，派生类继承了所有基类的函数与成员，要按照字节对齐来计算大小

```cpp
class A
{
public:
    char a;
    int b;
};
class B : A
{
public:
    short a;
    long b;
};
/*
B 按照顺序：char a, int b, short a, long b
根据字节对齐：A(4 + 4) + 4 + 4 = 24
*/
class C {
    A a;
    char c;
}
/*
根据字节对齐：A(4 + 4) + 4 = 12
*/
```

- 虚函数继承，不管是单继承还是多继承，都是继承了基类的 vptr

```cpp
class A1
{
    virtual void fun(){}
};
// 4

class A2
{
    char a;
    virtual void fun(){}
};
// 8

class D:public A1
{
};
// 4

class E:public A1
{
    char c;
};
// 4 + 4

class F:public A2
{
    char c;
};
// 4 + 4 + 4
```

- 虚继承，继承基类的 vbptr，并添加一个指向基类的虚基类指针 vfptr

```cpp
/*
vptr    虚函数指针
vtptr   需基类指针
*/

class A
{
	virtual void fun() {}
};
// 1 个 vptr
// 4

class B
{
	virtual void fun2() {}
};
// 1 个 vptr
// 4

class C : virtual public A, virtual public B
{
public:
	virtual void fun3() {}
};
// (A)1 个 vbptr +  (A)1 个 vptr + (B)1 个 vbptr +  (B)1 个 vptr
// sizeof(C) = 16

class C1 : public A, public B
{
public:
	virtual void fun3() {}
};
// (A)1 个 vptr + (B)1 个 vptr
// 8

class D : virtual public A {
	virtual void fun() {}
};
// 1 个 vbptr +  1 个 vptr
// 8   4 + 4

class D1 : A {
	virtual void fun() {}
};
// 1 个 vbptr
// 4

class E : virtual public A {
	virtual void fun() {}
};
// 1 个 vbptr +  1 个 vptr
// 8 

class F : public D, public E {
	virtual void fun() {}
};
// 1 个 vptr + (D)1 个 vbptr + (E)1 个 vbptr


class SuperBase {
public:
	int m_nValue;
	void Fun() { cout << "SuperBase1" << endl; }
	virtual ~SuperBase() {}
};
// 4 + 4

class Base1 : virtual public SuperBase {
public:
	virtual ~Base1() {}
};
// 4 + 4 + 4

class Base2 : virtual public SuperBase
{
public:
	virtual ~Base2() {}
};
// 12

class Der :public Base1, public Base2 {
public:
	virtual ~Der() {}
};
// 16

```