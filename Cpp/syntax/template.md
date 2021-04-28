# 模板

## 函数模板

函数模板在形式上分为模板、函数两部分

对函数的各种修饰 `inline`、`constexpr` 等需要加在函数声明上，而不是模板前

```cpp
template <class T>
inline T min(const T &, const T &);
```

模板参数列表可以包括：

|序号|名称|说明
|-|-|-|
1|非参数形参|已知的数据类型，如 `int N`、`int N = 1` 带默认值、`int ...N` 模板函数包
2|类型形参|`class T = [default]`、`class ...T` 模板参数包
3|模板模板形参|模板中有模板

函数模板就像是一种契约，任何满足该契约的类型都可以做为模板实参，而契约就是函数实现中模板实参需要支持的各种操作

函数模板本质上函数的重载，泛型只是简化了程序员的工作，让这些重载通过编译器来完成

在不知道模板参数的具体类型前，无法确定模板函数需要占用的栈大小（参数栈，局部变量）， 同时也不知道函数体中模板形参的各种操作如何实现，无法生成具体的函数

只有当用具体类型去替换模板参数时，才会生成具体函数，该过程叫做函数模板的实例化，所以相较普通函数，函数模板多了生成具体函数这一步

如果我们只是编写了函数模板，但不在任何地方使用它，则编译器不会为该函数模板生成任何代码

### 隐式实例化

当函数模板被调用，且在之前没有显式实例化时，即发生函数模板的隐式实例化

如果模板实参能从调用的语境中推导则不需要提供，效率较低

### 显示实例化

在函数模板定义后，可以通过显式实例化的方式告诉编译器生成指定实参的函数，在编译期间就会生成实例

显式实例化声明会阻止隐式实例化

在显式实例化时，只指定部分模板实参，则指定顺序必须自左至右依次指定，不能越过前参模板形参，直接指定后面的

## 可变模板

可变模板可以很好地做递归调用

```cpp
template<typename... A> class Car;  
//typename...就表示一个模板参数包。可以这么来实例化模板：
Car<int, char> car; 

template <class... T>
void f(T... args);  // T... 可变模板参数的展开
// 无法直接获取参数包 args 中的每个参数的，只能通过展开参数包的方式来获取参数包中的每个参数

sizeof...(args)  // 得到可变模板里元素的个数
```

### 泛化能力

```cpp
template <typename T, typename... Types> 
void print(cosnt T& arg, const Types&... args) {

}

template <typename... Types> 
void print( const Types&... args) {

}

/*
上面两个模板函数，
*/
```

### 递归函数方式展开函数包

需要提供一个参数包展开函数和一个重载的递归终止函数
```cpp
#include <iostream>
using namespace std;
//递归终止函数
void print() {
   cout << "empty" << endl;
}
//展开函数
template <class T, class... Args>
void print(T head, Args... rest) {
   cout << "parameter " << head << endl;
   print(rest...);
}

int main(void) {
   print(1,2,3,4);
   return 0;
}
```

### 逗号表达式展开从参数包

借助逗号表达式和初始化列表

```cpp
// 逗号表达式会按顺序执行逗号前面的表达式
d = (a = b, c);
// b 先赋值给 a，接着括号中的逗号表达式返回 c 的值，d == c

template <class T>
void printarg(T t)  // 不是递归终止函数，只是一个处理参数包中每个参数的函数
{
   cout << t << endl;
}

template <class ...Args>
void expand(Args... args)
{
   int arr[] = {(printarg(args), 0)...};  // 直接在这展开
}

expand(1,2,3,4);
```

### tuple 实现

```cpp
template<> class tuple <> {};
template<typename ... Values> class tuple;

template <typename Head, typename... Tail> 
class tuple<Head, Tail> : private tuple<Tail...> {
    typedef tuple<Tail...> inherited;
public:
    tuple() {}
    tuple(Head v, Tail... vtail) : m_head(v), inherited(vtail...) {}

    typename Head::type head() {return m_head;}
    inherited& tail()  {return *this;}

protected:
    Head m_head;
}
```