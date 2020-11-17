## 初始化列表

初始化列表可以初始化任意类型对象

## initializer_list<T>

stl 中的容器通过类模板 `std::initializer_list<T>` 来完成初始化列表的任意长度

`std::initializer_list<T>` 内部关联一个 `std::array<T, n>`，

自定义类只需要添加一个 `std::initializer_list<T>` 的构造函数，也可以拥有任意长度初始化的能力

```cpp
class Foo {
   std::vector<int> vec;
public:
   Foo(std::initializer_list<int> list) {
    for(auto ite = list.begin(); ite != list.end(); ++ite) {
        vec.push_back(*ite);
    }
   };
}
```

当自定义类没有 `std::initializer_list<T>` 的构造函数，编译器会将初始化列表里的元素逐一拆解调用

```cpp
template <typename T>
class complex<T> {
private:
    T n1;
    T n2;

public:
    complex(T t1, T t2) : n1(t1), n2(t2) {}
}

complex<double> cmp {0.1, 0.2};
```

`std::initializer_list<T>` 还可以用来传递同类型的数据集合

```cpp
void func(std::initializer_list<int>) {
    for(auto ite = list.begin(); ite != list.end(); ++ite) {
        std::cout<<*ite<<std::endl;
    }
}
```

`std::initializer_list<T1>` 内部并不负责保存初始化列表中元素，仅仅存储了列表中元素的引用，元素由背后的 `std::array` 存储

## 不允许类型收窄

不允许隐式类型转换过程中丢失精度

```cpp
float myarray[3]{1.0,2.0,3.0};
int myintarray[4]{};  // 把所有元素都设置为零，相当于自动执行了 memset

// 列表初始化禁止缩窄转换，即数据类型的精度不能向下自动转换
long plimit[]={25, 3.0};  //错误，3.0是float类型不能再初始化列表中转换为long类型
char slimit[]{ 'a', 1122345};//错误，1122345超出了char能表示的范围
char slimit2[]{'a',110};//合法，110在char的表示范围内
```