# 策略模式

利用策略模式来避免冗长的 `if-else` 或 `switch` 分支判断

定义一族算法类，将每个算法分别封装起来，让它们可以互相替换

策略模式解耦了策略的定义、创建、使用

## 策略定义

策略类的定义比较简单，包含一个策略接口和一组实现这个接口的策略类，所有的策略类都实现相同的接口，所以，客户端代码基于接口而非实现编程，可以灵活地替换不同的策略

```cpp
// 抽象算法类
class Strategy { 
public:
    virtual void AlgorithmInterface() = 0; // 算法方法
    virtual ~Strategy(){}
};

// 具体算法A
class ConcreteStrategyA : public Strategy { 
public:
    void AlgorithmInterface() {
        cout << "ConcreteStrategyA" << endl;
    }
};

// 具体算法B
class ConcreteStrategyB : public Strategy { 
public:
    void AlgorithmInterface() {
        cout << "ConcreteStrategyB" << endl;
    }
};
```

## 策略创建

策略模式会包含一组策略，在使用它们的时候，一般会通过 type 来判断创建哪个策略来使用

为了封装创建逻辑，我们需要对客户端代码屏蔽创建细节，可以把根据 type 创建策略的逻辑抽离出来，放到工厂类中

```cpp
// 没有状态的策略工厂
class ContextFactory { 
private:
    Strategy* strategy = nullptr;
public:
    ContextFactory() = default;

    ~ContextFactory() {}

    Strategy* GetContext(char c) { 
        switch(c) {
            case 'A': strategy = new ConcreteStrategyA(); break;
            case 'B': strategy = new ConcreteStrategyB(); break;
            default : strategy = NULL;
        }
    }
};

// 带状态的策略工厂
class ContextFactory { 
private:
    unordered_map<char, Strategy*> um;
public:
    ContextFactory() {
        um['A'] = new ConcreteStrategyA();
        um['B'] = new ConcreteStrategyB();
    }

    ~ContextFactory() {}

    Strategy* GetContext(char c) { return um[c]; }
};
```

## 策略使用

```cpp
int main() {
    ContextFactory factory;
    auto s = factory.GetContext('A');
    s.AlgorithmInterface();
    delete s;
    return 0;
}
```