# 装饰器模式

在不增加子类的情况下增强一个类的功能

装饰器类和原始类继承同样的父类，这样就可以对原始类嵌套多个装饰器类，起到增强类功能的作用

```cpp
struct Game {
    virtual ~Game() {}

    virtual void Play() = 0;
};

struct BasketBallDecorator : public Game {
    BasketBallDecorator() {}
    BasketBallDecorator(Game* game) { game_ = game; }
    void Play() override {
        std::cout << "play basketball \n";
        if (game_) game_->Play();
    }

   private:
    Game* game_;
};

struct SocketBallDecorator : public Game {
    SocketBallDecorator() {}
    SocketBallDecorator(Game* game) { game_ = game; }
    void Play() override {
        std::cout << "play SocketBall \n";
        if (game_) game_->Play();
    }

   private:
    Game* game_;
};

int main() {
    Game* ball = new BasketBallDecorator();
    ball = new SocketBallDecorator(ball);  // 暂时忽略内存泄漏
    ball->Play();
    return 0;
}
```

装饰器模式和代理模式有个重要区别就是装饰器模式是对一个类的增强，附加的是跟原始类有关的功能，而代理模式附加的是与原始类无关的额外功能

装饰器模式关注于在一个对象上动态的添加方法，然而代理模式关注于控制对对象的访问

用代理模式，代理类（proxy class）可以对它的客户隐藏一个对象的具体信息，在一个代理类中创建一个对象的实例

使用代理模式，代理和真实对象之间的的关系通常在编译时就已经确定了，而装饰者能够在运行时递归地被构造