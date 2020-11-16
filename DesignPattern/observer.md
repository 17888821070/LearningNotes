# 观察者模式

观察者模式也被称为发布订阅模式

在对象之间定义一个一对多的依赖，当一个对象状态改变的时候，所有依赖的对象都会自动收到通知

被依赖的对象叫作被观察者（Observable），依赖的对象叫作观察者（Observer）

```cpp

// 抽象观察者，为所有具体观察者定义一个接口，在得到主题的通知时更新自己，这个接口叫更新接口
class Observer { 
public:
	virtual void Update() = 0;
};

// 主题或抽象通知者，一般用一个抽象类或者接口实现
class Subject { 
private:
	list<Observer* > observers;
public:
	void Attach(Observer* observer) { observers.push_back(observer); } // 增加观察者
	void Detach(Observer* observer) { observers.remove(observer); } // 移除观察者
	void Notify() { // 通知
		for (auto observer = observers.begin(); observer != observers.end(); observer++) {
			(*observer)->Update();
		}
	}
};

// 具体主题或具体通知者，将有关状态存入具体观察者对象，在具体主题的内部状态改变时，给所有登记过的观察者发送通知
class ConcreteSubject : public Subject { 
private:
	string subjectState;
public:
	string GetState() { return subjectState; }
	void SetState(string state) { subjectState = state; }
};

// 具体观察者，实现抽象观察者角色所要求的更新接口，以便使本身的状态与主题状态相协调
class ConcreteObserver : public Observer { 
private:
	string name;
	ConcreteSubject* subject;
	string observerState;
public:
	ConcreteObserver(ConcreteSubject* s, string n) {
		subject = s;
		name = n;
	}
	void Update() {
		observerState = subject->GetState();
		cout << "observer: " << name << ", its new state is: " << observerState << endl;
	}
	string GetState() { return observerState; }
	void SetState(string state) { observerState = state; }
};

int main() {
	ConcreteSubject* s = new ConcreteSubject();
	s->SetState("ABC");
	ConcreteObserver* ox = new ConcreteObserver(s, "X");
	ConcreteObserver* oy = new ConcreteObserver(s, "Y");
	ConcreteObserver* oz = new ConcreteObserver(s, "Z");
	s->Attach(ox);
	s->Attach(oy);
	s->Attach(oz);
	s->Notify();
	// observer: X, its new state is: ABC
	// observer: Y, its new state is: ABC
	// observer: Z, its new state is: ABC
	s->SetState("XYZ");
	s->Notify();
	// observer: X, its new state is: XYZ
	// observer: Y, its new state is: XYZ
	// observer: Z, its new state is: XYZ
	delete ox;
	delete oy;
	delete oz;
	delete s;
	return 0;
}
```

观察者模式也分为同步阻塞的实现方式和异步非阻塞的实现方式，有进程内的实现方式也有跨进程的实现方式

在每个 `Update()` 函数中开线程实现异步，但有线程创建、销毁的开销，而且并发数不受控制；利用线程池，在 `Notify()` 函数中将每个 `Update()` 放入线程池，但线程池、异步执行逻辑都耦合在了 `Notify()` 函数中，增加了代码的维护成本

使用事件总线 EventBus，

跨进程中可以使用 RPC 来执行 `Update()`，也可以使用消息队列实现

观察者需要注册到被观察者中，被观察者需要依次遍历观察者来发送消息；使用 MQ 时，被观察者和观察者解耦更加彻底，两部分的耦合更小，被观察者完全不感知观察者，观察者也完全不感知被观察者，被观察者只管发送消息到消息队列，观察者只管从消息队列中读取消息来执行相应的逻辑