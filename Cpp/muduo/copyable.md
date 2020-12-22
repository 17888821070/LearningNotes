# Copyable

```cpp
/// A tag class emphasises the objects are copyable.
/// The empty base class optimization applies.
/// Any derived class of copyable should be a value type.
class copyable {
protected:
    copyable() = default;
    ~copyable() = default;
};

class noncopyable {
public:
    noncopyable(const noncopyable&) = delete;
    void operator=(const noncopyable&) = delete;

protected:
    noncopyable() = default;
    ~noncopyable() = default;
};
```

基本思想是把构造函数和析构函数设置 protected 权限，这样子类可以调用，但是外面的类不能调用，
那么当子类需要定义构造函数的时候不至于通不过编译

禁止编译器自动生成把 copy 构造函数和 copy 赋值函数，这就意味着除非子类定义自己的 copy 构造和赋值函数，否则在子类没有定义的情况下，外面的调用者是不能够通过赋值和 copy 构造等手段来产生一个新的子类对象的