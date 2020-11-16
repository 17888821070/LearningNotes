# 原型模式

如果对象的创建成本比较大，而同一个类的不同对象之间差别不大（大部分字段都相同），在这种情况下，我们可以利用对已有对象（原型）进行复制（或者叫拷贝）的方式来创建新对象，以达到节省创建时间的目的

可以利用原型模式，从其他已有对象中直接拷贝得到，而不用每次在创建新对象的时候，都重复执行这些耗时的操作

原型模式实现的关键就是实现 `Clone()` 函数，分为深拷贝和浅拷贝


```cpp
struct Animal {
    virtual ~Animal() {}

    virtual Animal* Clone() = 0;
    virtual void ShowName() = 0;
};

struct Tiger : public Animal {
    Animal* Clone() override { return new Tiger(); }

    void ShowName() override { std::cout << "Tiger" << std::endl; }
};

int main() {
    Animal* animal = new Tiger();
    animal->ShowName();
    Animal* animal_copy = animal->Clone();  // 想要一个和animal完全相同的实例
    animal_copy->ShowName();
    return 0;
}
```

原型模式还有个重要意义：当一个基类指针指向某个子类对象时，这时如果想要拷贝这个子类对象是比较困难的，因为只通过一个基类指针我们不知道该指针究竟指向了什么类型的对象，即无法调用相应的构造函数，通过 `typeid` 加 `switch` 貌似代价太大，所以可以使用此原型模式