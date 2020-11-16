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
