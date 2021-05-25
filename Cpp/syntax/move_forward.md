# 右值相关

## 左值与右值

- 左值是在作用域内表达式运行结束后依然存在的持久对象，它有明确地址能够被访问到

- 右值指表达式结束后不再存在的临时对象，且进一步划分为纯右值和将亡值

- 纯右值指字面量或求值结果相当于字面量或匿名临时变量、非引用返回的临时变量等，它没有地址可供访问

- 将亡值指即将被销毁却能够被移动的值，它有地址但是仅能被编译器操作，程序不可访问，它的资源可以被复用

```cpp
int a = 3;  // 左值

int setValue() {
    a = 4;  // 4 是字面量，纯右值
    return a;  // 返回结果为将亡值
}
```

## move

`#include <utility>`

`std::move()` 执行到右值的无条件转换，返回右值引用本质上还是靠 `static_cast<T&&>` 完成的

```cpp
template<typename T>
typename remove_reference<T>::type && move(T&& t)
{
    return static_cast<typename remove_reference<T>::type &&>(t);
}

std::move(std::string("111"))
// T 是 std::string
// 返回 std::string&& 形参 std::string&&

std::string s1("111")
std::move(s1)
// T 是 std::string&
// 返回 std::string&& 形参 std::string&&
```

 右值引用类型是独立于值的，一个右值引用参数作为函数的形参，在函数内部再转发该参数的时候它已经变成一个左值，并不是他原来的类型；需要一种方法能够按照参数原来的类型转发到另一个函数，这种转发类型称为完美转发

`std::forward()` 实现了参数在传递过程中保持其值属性的功能，即若是左值，则传递之后仍然是左值，若是右值，则传递之后仍然是右值，返回值为 `static_cast<T&&>(t)`

`std::move` 执行到右值的无条件转换，`std::forward` 只有在它的参数绑定到一个右值上的时候才转换它的参数到一个右值

## forward

`std::forward` 不仅可以保持左值或者右值不变，同时还可以保持 `const`、`Lreference`、`Rreference`、`volatile` 等属性不变

```cpp
void PrintV(int &t) {
    cout << "lvalue" << endl;
}

void PrintV(int &&t) {
    cout << "rvalue" << endl;
}

template<typename T>
void Test(T &&t) {
    PrintV(t);
    PrintV(std::forward<T>(t));

    PrintV(std::move(t));
}

int main() {
    Test(1); // lvalue rvalue rvalue
    int a = 1;
    Test(a); // lvalue lvalue rvalue
    Test(std::forward<int>(a)); // lvalue rvalue rvalue
    Test(std::forward<int&>(a)); // lvalue lvalue rvalue
    Test(std::forward<int&&>(a)); // lvalue rvalue rvalue
    return 0;
}
```

## 模板类型推导

- 形参是指针或引用类型，但不是万能引用

```cpp
// 模板型别推导的最终结果会忽略掉实参的引用部分（如果有）
template<typename T>
void f(T& param);

int x = 10;
const int cx = x;
const int& rx = x;

f(x);  // param 为 int&
f(cx); // param 为 const int&
f(rx); // param 为 const int&（实参的 & 被忽略了）
// 以上的形参是左值引用，当改成右值引用时，情况一样
```
- 形参是万能引用，既不是左值引用也不是右值引用的引用，并且它最终既可以做左值引用也可以做右值引用，形如 `T&&`，只要涉及型别推导，那么它就是一个万能引用，否则就是右值引用

引用折叠规则：如果间接的创建一个引用的引用，则这些引用就会折叠

1. `X& &&`、 `X&& &` 都折叠成 `X&`
2. `X&& &&` 折叠为 `X&&`

```cpp
void f(Widget&& param);  // param 是右值引用，因为它的类型已经确定，不涉及型别推导

Widget&& var1 = Widget();  // 同上

auto&& var2 = var1;  // 万能引用

template<typename T>
void f(std::vector<T>&& param);  // 右值引用
    
template<typename T>
void f(T&& param);  // 万能引用


template<typename T>
void f(T&& param);  // param 现在是个万能引用

int x = 10;  // 同前
const int cx = x;  // 同前
const int& rx = x;  // 同前

f(x);  // 因为 x 是左值，所以 param 为 int&，T = int&
f(cx);  // 因为 x 是左值，所以 param 为 const int &，T = const int&
f(rx);  // 因为 x 是左值，所以 param 为 const int & ，T = const int&（实参的&被忽略了）

f(10);  // 因为 10 是右值，所以 param 为 int &&，T = int
```

万能引用的作用就是当你传入左值，那么它最终就是左值引用，传入右值，那么它就是右值引用

右值引用的特殊类型推断规则：当将一个左值传递给一个参数是右值引用的函数，且此右值引用指向模板类型参数 `T&&` 时，编译器推断模板参数类型为实参的左值引用

```cpp
template<typename T> 
void f(T&&);

int i = 42;
f(i)
// 模板参数类型 T 将推断为 int& 类型，而非 int
// f将被实例化为: void f<int&>(int&)
```

- 既不是指针也不是引用

```cpp
/*
首先，模板型别推导会像之前的情况一样，忽略掉实参的引用部分

其次，按值传递意味着 param 对象将是实参的一个副本，由于对副本的修改并不会影响原有对象，所以如果原有对象（实参）具有 const、valatile 的修饰，也会一并忽略掉
*/
template<typename T>
void f(T param);//param是实参的一个副本

int x = 10; //同前
const int cx = x;//同前
const int& rx = x;//同前

f(x);//param是int
f(cx);//param是int，忽略掉了const
f(rx);//param是int，忽略掉了const和&
```