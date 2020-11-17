# 可变模板

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

## 泛化能力

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

## 递归函数方式展开函数包

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

## 逗号表达式展开从参数包

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

## tuple 实现

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