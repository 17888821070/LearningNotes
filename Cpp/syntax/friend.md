# 友元

## 概述

友元提供了一种普通函数或者类成员函数访问另一个类中的私有或保护成员的机制

- 友元关系没有继承性，假如类 B 是类 A 的友元，类 C 继承于类 A，那么友元类 B 是没办法直接访问类 C 的私有或保护成员

- 友元关系没有传递性，假如类 B 是类 A 的友元，类 C 是类 B 的友元，那么友元类 C 是没办法直接访问类A的私有或保护成员，也就是不存在友元的友元这种关系

友元提高了程序的运行效率，破坏了类的封装性和数据的透明性

## 友元函数

赋予友元函数与类的成员函数相同的访问权限

- 创建友元函数第一步是将其原型放在类声明中，并在原型声明前加入关键字 `friend`，`friend 返回类型 函数名(形参列表)` 友元函数可以重载运算符，友元函数虽然在类声明中声明的，但它不是成员函数，不能使用成员运算符来调用；友元函数与成员函数的访问权限相同

- 编写函数定义，不需要使用 `类::`，且不需要关键字 `friend`

- 对于非成员重载运算符函数来说，运算符表达式左边的操作数对应于运算符函数的第一个参数，运算符表达式右边的操作数对应于运算符函数的第二个参数

```cpp
class A
{
public:
    A(){};
    
};
A a;
cout << a;  // 重载不了<<  因为cout不是类的实例对象 

class A
{
public:
    A(){};
    friend void operator<<(ostream & os, const Time & t);
};

void operator<<(ostream & os, const Time & t)
{
    
}
A a;
cout << a;  // 重载<<函数
```

## 友元类

友元类的声明在该类的声明中，而实现在该类外

```cpp
class A
{
public:
    A(int _a):a(_a){};
    friend class B;
private:
    int a;
};

class B
{
public:
    int getb(A ca) {
        return  ca.a; 
    };
};

int main() 
{
    A a(3);
    B b;
    cout<<b.getb(a)<<endl;
    return 0;
}
```

## 友元成员函数

将一个类的成员函数声明为另一个类的友元

```cpp
class A {
friend B::func(A& a);
private:
    int a;
    int b;
}

class B {
public:
    void func(A& a);
}
```