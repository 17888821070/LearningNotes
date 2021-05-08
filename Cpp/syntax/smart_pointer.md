# 智能指针

## auto_ptr

C++ 17 中移除

```cpp
template<class T>
class AutoPtr  // 模拟实现 auto_ptr
{
public:
	AutoPtr(T* tmp = nullptr)
		:_ptr(tmp)
	{}

	~AutoPtr()
	{
		if (_ptr)
			delete _ptr;
	}

	T& operator*()
	{
		return *_ptr;
	}
		
	T* operator->()
	{
		return _ptr;
	}

	AutoPtr(AutoPtr<T>& _tmp)  // 拷贝构造
		:_ptr(_tmp._ptr)
	{
		_tmp._ptr = nullptr;
	}

	AutoPtr<T>& operator=(AutoPtr<T>& _tmp)  // 赋值运算符重载，记得要释放原空间
	{
		if (_tmp._ptr != _ptr)
		{
			if (_ptr)
				delete _ptr;
			_ptr = _tmp._ptr;
			_tmp._ptr = nullptr;
		
		}
		return *this;
	}
private:
	T* _ptr;
};
```
- 在拷贝/赋值过程中，直接剥夺原对象对内存的控制权，转交给新对象，然后再将原对象指针置为 `nullptr`，新对象原管理的内存会被释放掉

- 一个指针变量指向的空间不能由两个 `auto_ptr` 管理，不然会析构两次，使程序崩溃

- `auto_ptr` 不能用来管理数组，析构函数中用的是 `delete`

- 成员函数 `release` 只会把指针指针的指针赋值为空，而不去释放内存；成员函数 `reset` 会释放内存

- 将 `auto_ptr` 作为函数参数时，传进去的时候转移了控制权，但函数返回时并没有将控制权移交，所以经过函数调用的 `auto_ptr` 失去了内存的控制权

## unique_ptr

`unique_ptr` 是一个独享所有权的智能指针

```cpp
template<class T>
class UniquePtr // 模拟实现unique_ptr
{
public:
	UniquePtr(T* tmp = nullptr)
		:_ptr(tmp)
	{}

	~UniquePtr()
	{
		if (_ptr)
			delete _ptr;
	}

	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}
	
	
private:
	UniquePtr(const UniquePtr<T>& tmp) = delete;
	UniquePtr<T>& operator=(const UniquePtr<T>& tmp) = delete;
	T* _ptr;
};
  // = delete 禁止使用编译器默认生成的函数
```
直接把拷贝构造/赋值函数弄成 `delete`，即将这两个函数定义成已删除的函数，任何试图调用它的行为将产生编译期错误，不允许 `unique_ptr` 进行拷贝、赋值，但是可以进行移动构造和移动赋值

## shared_ptr

```cpp
template<class T>
class SharedPtr // 模拟实现shared_ptr
{
public:
	SharedPtr(T* tmp = nullptr)
		:_ptr(tmp)
		,count(new int(1))
	{}

	~SharedPtr()
	{
		if (--(*count) == 0)
		{
			delete _ptr;
			delete count;
		}
	}

	T& operator*()
	{
		return *_ptr;
	}

	T* operator->()
	{
		return _ptr;
	}
	
	SharedPtr(SharedPtr<T>& tmp)
		:_ptr(tmp._ptr)
		,count(tmp.count)
	{
		(*count)++;
	}

	SharedPtr<T>& operator=(SharedPtr<T>& tmp)
	{
		if (_ptr != tmp._ptr)  // 排除自己给自己赋值的可能
		{
			// 先要判断原来的空间是否需要释放
			if (--(*count) == 0)
			{
				delete _ptr;
				delete count;
			}
			_ptr = tmp._ptr;
			count = tmp.count;
			(*count)++;
		}
		return *this;  // 考虑连等的可能
	}
private:
	T* _ptr;
	int* count;
};
```
- 通过引用计数的方式来实现多个 `shared_ptr` 对象之间共享资源;  `shared_ptr` 在其内部，给每个资源都维护了着一份计数，用来记录该份资源被几个对象共享;在对象被销毁时(也就是析构函数调用)，就说明自己不使用该资源了，对象的引用计数减一;如果引用计数是 0，就说明自己是最后一个使用该资源的对象，必须释放该资源;如果不是 0，就说明除了自己还有其他对象在使用该份资源，不能释放该资源，否则其他对象就成野指针了

- 在多线程程序下，多个线程都去访问 `shared_ptr` 管理的空间，如果线程是并行的，那么引用计数会可能发生错误；C++11 定义一个互斥锁，对所有引用计数 `++` 或者 `–-` 进行加锁处理，这样就能保障引用计数在操作的过程中能够不被切出去，因此 `shared_ptr` 本身引用计数是线程安全的

- `shared_ptr` 内部有两个指针，这两个指针的拷贝在多线程下是不安全的

- `shared_ptr<void>` 可以持有任何对象，并且安全释放


```cpp
// 指定删除器来释放内存
void deleter(int * pt) {
	delete [] pt;
}

shared_ptr<int> sp(new int [10], deleter);
```

- 循环引用问题

```cpp
struct Node
{
	int _data;
	shared_ptr<Node> _prev;
	shared_ptr<Node> _next;

	~Node(){ cout << "~Node()" << endl; }
};
int main()
{
	shared_ptr<Node> node1(new Node);
	shared_ptr<Node> node2(new Node);
	//ues_count == 1
	cout << node1.use_count() << endl;
	cout << node2.use_count() << endl;

	node1->_next = node2;
	node2->_prev = node1;

	//ues_count == 2
	cout << node1.use_count() << endl;
	cout << node2.use_count() << endl;
	return 0;
}
```
node1 和 node2 两个智能指针对象指向两个节点，引用计数变成 1，不需要手动 `delete`；node1 的_next 指向 node2 ，node2 的 _prev 指向 node1，它们的引用计数变成 2；node1 和 node2 析构，引用计数减到 1，但是 node1->_next 还指向 node2 节点，node2->_prev 还指向 node1；当 _next 析构时，node1 释放资源，当 _prev 析构时node2 释放资源。但是 _next 属于 node1 的成员，node1 释放了，_next 才会析构，而 node1 由 _prev 管理，_prev 属于 node2 成员，所以这就叫循环引用，谁也不会释放


## weak_ptr

为解决 `shared_ptr` 的循环引用问题，引出 `weak_ptr` 智能指针，`weak_ptr` 在指向被监视的 `shared_ptr` 后，并不会使被监视的引用计数增加，且当被监视的对象析构后就自动失效

- 必须通过 `shared_ptr` 来共享内存

- 没有重载 `opreator*` 和 `->` 操作符，也就意味着即使分配到对象，他也没法使用该对象，使用 `lock()` 可以得到 `shared_ptr` 或直接转化为 `shared_ptr`

```cpp
weak_ptr<A> wptr;
shared_ptr<A> obj(wptr.lock());
if(obj) {
	// 转化成功
}
else {
	// 转化失败，wptr 指向的对象已析构
}
```

## make_shared、make_unique

`std::make_shared` 模板函数，可以返回一个指定类型的 `std::shared_ptr`

`std::make_unique` 模板函数，可以返回一个指定类型的 `std::unique_ptr`

```cpp
shared_ptr<string> p1 = make_shared<string>(10, '9');  
 
shared_ptr<string> p2 = make_shared<string>("hello");  
 
shared_ptr<string> p3 = make_shared<string>(); 
```

- `std::make_xxxxx` 比起直接使用 `new`，效率更高

- 异常安全，发生异常时仍释放资源

- 构造函数是保护或私有时，无法使用 `std::make_xxxx`

- 对象的内存可能无法及时回收

- `std::make_xxxxx` 无法指定删除器

## enable_shared_from_this

`enable_shared_from_this` 是一个以其派生类为模板类型实参的基类模板，继承它可将 `this` 转化成 `shared_ptr`

```cpp
class B : public std::enable_shared_from_this<B> {

}

shared_ptr<B> ptr = make_shared<B>();
```

派生类不再是栈上的资源，而是堆上的资源，并且由 `shared_ptr` 管理声明周期

使用 `shared_from_this()` 获取 `this`

`shared_from_this()` 不能在构造函数中调用，因为对象还没转交给 `shared_ptr`