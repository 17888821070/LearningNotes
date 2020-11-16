# inline 内联

如果一个函数是内联的，那么在编译时编译器会把该函数的代码副本放置在每个调用该函数的地方；对内联函数进行任何修改，都需要重新编译函数的所有客户端，因为编译器需要重新更换一次所有的代码，否则将会继续使用旧的函数

如果想把一个函数定义为内联函数，则需要在函数名前面放置关键字 inline，在调用函数之前需要对函数进行定义

如果已定义的函数多于一行，编译器会忽略 inline 限定符

在类定义中的定义的函数都是内联函数，即使没有使用 inline 说明符

```cpp
class A {
public:
    // 类中定义即隐式内联函数
    void func(int x, int y) {

    };
    // 声明后要想成为内联函数，必须在定义处加 inline 
    void f1(int x);  
}

inline void A::f1(int x) {

}

// 普通函数成为内联函数
inline int max(int x, int y) {
    return x > y ? x : y;
}

int main() {
    cout<<max(1, 2)<<endl;
    // 编译器会将 max() 的代码复制过来
    // cout<<x > y ? x : y<<endl;
    return 0;
}
```