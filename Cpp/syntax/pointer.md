# 指针相关

## 二级指针
```cpp
int data[3][4] = {{1, 2, 3, 4}, {1, 2, 3, 4}, {1, 2, 3, 4}};
// data 是一个数组名，该数组有 3 个元素；每一个元素本身是一个数组，由 4 个 int 值组成
// data 的类型是一个指向由 4 个 int 组成的数组的指针，即 int (*data)[4]

int ** p;  // 二级指针
p = (int**)data;  // 强制类型转换
```


## 数组指针和指针数组
```cpp
// 符号优先级： () > [] > *
int (*p)[n];  
// 根据优先级，先看()，则 p 是一个指针；这个指针指向一个一维数组，数组长度为 n，即数组指针

int *p[n];
// 根据优先级，先看 []，则 p 是一个数组，再结合 *，这个数组的元素是指针类型即指针数组，共 n 个元素
```

### 数组指针

```cpp
int main()
{
    int a[5] = {1, 2, 3, 4, 5};
    int (*p)[5];
    p = &a;

    cout << a << endl;        // 输出 a[0] 的地址
    cout << p << endl;        // 输出数组的地址，数组的地址用数组首元素地址标识
    cout << *p << endl;       // *p 表示数组本身，用数组的首元素地址来标识一个数组
    cout << &a[0] << endl;    // 输出 a[0] 的地址
    cout << &a[1] << endl;    // 输出 a[1] 的地址
    cout << p[0] << endl;     // 输出 a[0] 的地址
    cout << p[1] << endl;     // 输出 a + 5 * siezof(int) 的地址
    cout << **p << endl;      // 输出 1
    cout << *p[0] << endl;    // 输出 1
    cout << *p[1] << endl;    // 输出地址为 a + 5 * siezof(int) 的值

    getchar();
    return 0;
}
```

### 指针数组
```cpp
int main()
{
    int a = 1;
    int b = 2;
    int *p[2];

    p[0] = &a;
    p[1] = &b;

    cout << p[0] << endl;   // 输出 a 的地址
    cout << &a << endl;     
    
    cout << p[1] << endl;   // 输出 b 的地址
    cout << &b << endl;     

    cout << *p[0] << endl;  // 输出 a
    cout << *p[1] << endl;  // 输出 b

    getchar();
    return 0;
}
```

### 综合事例

```cpp
int main()
{
	int a[2][3] = { {0, 1, 2}, {3, 4, 5} };  // 2 行 3 列的二维整型数组
	int(*p)[3];  // 数组指针，指向含有 3 个元素的一维数组
	int * q[2];  // 指针数组，一个数组内存放 2 个指针变量
 
	p = a;
	q[0] = a[0]; // 指针指向了第一个数组元素
	q[1] = a[1];
 
	//输出第 1 行第 2 列的值
	printf("%d\n", a[1][2]);  // 5
 
	printf("%d\n", *(p[1] + 2));  // 5
	printf("%d\n", *(*(p + 1) + 2));  // 5
	printf("%d\n", (*(p + 1))[2]);  // 5
	printf("%d\n", p[1][2]);  // 5
 
	printf("%d\n", *(q[1] + 2));  // 5
	printf("%d\n", *(*(q + 1) + 2));  // 5
	printf("%d\n", (*(q + 1))[2]);  // 5
	printf("%d\n", q[1][2]);  // 5
 
    return 0;
}
```

## 函数指针

```cpp
char glFun(int a) {
    return a;
}

// 形式 1：返回类型(*函数名)(参数表) 
int main() {
    char (*pFun)(int);  // 定义了一个变量 pFun
    pFun = glFun;
    pFun(1);
    (*pFun)(2);
    return 0;
}

// 形式 2：使用 typedef
typedef char (*fun_t_p)(int);
typedef char (&fun_t_r)(int);
typedef char fun_t_t(int);

int main() {
    fun_t_p f_t_p = glFun;
    f_t_p(1);
    (*f_t_p)(2);

    fun_t_r f_t_r = glFun;
    f_t_r(1);
    (*f_t_r)(2);

    fun_t_t* f_t_t = glFun;
    f_t_t(1);
    (*f_t_t)(2);
    return 0;
}

// 形式 3：使用 using
using fun_u_p = char(*)(int);
using fun_u_r = char(&)(int);
using fun_u_t = char(int);

int main() {
    fun_u_p f_u_p = glFun;
    f_u_p(1);
    (*f_u_p)(2);

    fun_u_r f_u_r = glFun;
    f_u_r(1);
    (*f_u_r)(2);

    fun_u_t* f_u_t = glFun;
    f_u_t(1);
    (*f_u_t)(2);
    return 0;
}

// 类成员函数

// 静态方法函数指针语法
void (*ptrStaticFun)() = &ClassName::staticFun;
// 成员方法函数指针语法
void (ClassName::*ptrNonStaticFun)() = &ClassName::nonStaticFun;

class MyClass {
public:
	static int FunA(int a, int b) {
		cout << "call FunA" << endl;
		return a + b;
	}

	void FunB() {
		cout << "call FunB" << endl;
	}

	void FunC() {
		cout << "call FunC" << endl;
	}

	int pFun1(int(*p)(int, int), int a, int b) {
		return (*p)(a, b);
	}

	void pFun2(void (MyClass::* nonstatic)()) {
		(this->*nonstatic)();
	}
};

typedef void(MyClass::* f_t)();
using f_u = void(MyClass::*)();

int main() {
	MyClass* obj = new MyClass;
	// 静态函数指针的使用
	int(*pFunA)(int, int) = &MyClass::FunA;
	cout << pFunA(1, 2) << endl;

	// 成员函数指针的使用
	void (MyClass::*pFunB)() = &MyClass::FunB;
	(obj->*pFunB)();

	// 通过 pFun1 只能调用静态方法
	obj->pFun1(&MyClass::FunA, 1, 2);

	// 通过 pFun2 就是调用成员方法
	obj->pFun2(&MyClass::FunB);
	obj->pFun2(&MyClass::FunC);

	delete obj;
	return 0;
}
```