# 代理模式

## 原理和实现

代理模式在不改变原始类（或叫被代理类）代码的情况下，通过引入代理类来给原始类附加功能

代理模式中通常有三个角色，一个是抽象产品角色，一个是具体产品角色，另一个是代理产品角色，代理模式附加的是跟原始类无关的功能

```cpp
// Subject 类定义了 RealSubject 和 Proxy 的公用接口
// 这样就在任何使用 RealSubject 的地方都可以用 Proxy
class Subject {
public:
    virtual void Request() = 0;
    virtual ~Subject() {}
};

// RealSubject 类，定义了 Proxy 所代表的真实实体
class RealSubject : public Subject {
public:
    void Request() {
        cout << "RealSubject" << endl;
    }
};

// Proxy 类，保存一个引用使得代理可以访问实体
// 并提供一个与 Subject 的接口相同的接口，这样代理就可以用来代替实体
class Proxy : public Subject {
private:
    RealSubject* realSubject;
public:
    void Request() {
        if (realSubject == NULL)
            realSubject = new RealSubject();
        realSubject->Request();
    }
};
```

## 动态代理

静态代理需要针对每个类都创建一个代理类，并且每个代理类中的代码都有点像模板式的重复代码，增加了维护成本和开发成本

动态代理不事先为每个原始类编写代理类，而是在运行的时候动态地创建原始类对应的代理类，然后在系统中用代理类替换掉原始类

## 应用场景

- 业务系统的非功能性需求开发：将非功能性需求与业务功能解耦，放到代理类中统一处理，让程序员只需要关注业务方面的开发

- 在 RPC 的应用：RPC 框架也可以看作一种代理模式