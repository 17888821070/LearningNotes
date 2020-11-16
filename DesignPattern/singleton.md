# 单例模式

通过单例模式可以保证系统中某个类只有一个实例，并提供一个访问它的全局访问点，该实例被所有程序模块共享

单例类只能有一个实例，它必须自行创建这个实例，它必须自行向整个系统提供这个实例

实现：
- 单例模式的类只提供私有的构造函数
- 类定义中含有一个该类的静态私有对象
- 该类提供一个静态的共有的函数用于创建或获取它本身的静态私有对象

优点：
- 在内存中只有一个对象,节省内存空间
- 避免频繁的创建销毁对象,可以提高性能
- 避免对共享资源的多重占用
- 可以全局访问

虽然数量很少,但如果每次对象请求引用时都要检查是否存在类的实例,将仍然需要一些开销,这个问题可以通过静态初始化解决此问题


## 饿汉式单例（空间换时间）

在单例类被加载时就实例化一个对象交给自己的引用
```cpp
class CSingleton {  
public:  
    static CSingleton* GetInstance() {  
         return m_pInstance;  
    }  
private:  
    CSingleton(){};  
    CSingleton(CSingleton &){} = delete;  // 禁止拷贝
    CSingleton & operator=(const CSingleton &){} = delete;  // 禁止赋值
   
    static CSingleton * m_pInstance;  
}

CSingleton * CSingleton::m_Instance = new CSingleton;
```

## 有缺陷的懒汉式单例（时间换空间）

在调用取得实例方法的时候才会实例化对象

```cpp
class CSingleton {  
public:  
    static CSingleton* GetInstance() {  
        if ( m_pInstance == nullptr ) {
            m_pInstance = new CSingleton(); 
        } 
        return m_pInstance;  
    }  
private:  
    CSingleton(){};  
    CSingleton(CSingleton &){} = delete;  // 禁止拷贝
    CSingleton & operator=(const CSingleton &){} = delete;  // 禁止赋值
   
    static CSingleton * m_pInstance;  
}

CSingleton * CSingleton::m_Instance = nullptr;
```
`GetInstance()` 使用懒惰初始化，也就是说它的返回值是当这个函数首次被访问时被创建的

所有 `GetInstance()` 之后的调用都返回相同实例的指针

对 `GetInstance()` 稍加修改，这个设计模板便可以适用于可变多实例情况，如一个类允许最多五个实例

### 缺陷：
- 线程安全的问题：当多线程获取单例时有可能引发竞态条件；第一个线程在 `if` 中判断 `m_Instance` 是空的，于是开始实例化单例；同时第 2 个线程也尝试获取单例，这个时候判断 `m_Instance` 还是空的，于是也开始实例化单例；这样就会实例化出两个对象,这就是线程安全问题的由来，解决办法：加锁
- 内存泄漏：类中只负责 `new` 出对象，却没有负责 `delete` 对象，因此只有构造函数被调用，析构函数却没有被调用；因此会导致内存泄漏；解决办法：使用共享指针；

```cpp
class CSingleton {  
public:  
    static std::shared_ptr<CSingleton> GetInstance() {  
        if ( m_pInstance == nullptr ) {
            std::lock_guard<std::mutex> lk(m_mutex);
            if(m_pInstance == nullptr) {
                m_pInstance = std::shared_ptr<CSingleton>(new CSingleton); 
            }
        }
        return m_pInstance;  
    }  
private:  
    CSingleton(){};  
    CSingleton(CSingleton &){} = delete;  // 禁止拷贝
    CSingleton & operator=(const CSingleton &){} = delete;  // 禁止赋值
    
    static std::shared_ptr<CSingleton> m_pInstance;  
    static std::mutex m_mutex;
}

std::shared_ptr<CSingleton> CSingleton::m_pInstance = nullptr;
std::mutex CSingleton::m_mutex;

// 使用互斥量来达到线程安全。这里使用了两个if判断语句的技术称为双检锁
// 好处是，只有判断指针为空的时候才加锁，避免每次调用GetInstance()都加锁
// 隐患：析构函数是 public 的，能被外界 delete；因为构造函数是私有的，不能使用 make_shared
// 因为 CPU 的指令重排，可能会导致对象被 new 出来并赋值给 instance，但还没来得及初始化

class CSingleton {  
public:  
    static std::shared_ptr<CSingleton> GetInstance() {  
        if (m_pInstance == nullptr) {
                std::lock_guard<std::mutex> lk(m_mutex);
                if(m_pInstance == nullptr) {
                    m_pInstance = std::shared_ptr<CSingleton>(new CSingleton(), [](CSingleton * p){delete p}); 
                }
            }
        return m_pInstance;  
    }  
private:  
    CSingleton(){};  
    ~CSingleton(){};
    CSingleton(CSingleton &){} = delete;  // 禁止拷贝
    CSingleton & operator=(const CSingleton &){} = delete;  // 禁止赋值
    
    static std::shared_ptr<CSingleton> m_pInstance;  
    static std::mutex m_mutex;
}

std::shared_ptr<CSingleton> CSingleton::m_pInstance = nullptr;
std::mutex CSingleton::m_mutex;
```

### 局部静态变量型懒汉式单例 
(magic static)C++11
```cpp
class CSingleton {  
public:  
    static CSingleton & GetInstance() {  
        static CSinleton instance;  
        // 如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束
        // 这样保证了并发线程在获取静态局部变量的时候一定是初始化过的，所以具有线程安全性
        return instance;
    }  
private:  
    CSingleton(){};  
    ~CSingletion(){};
    CSingleton(CSingleton &){} = delete;  // 禁止拷贝
    CSingleton & operator=(const CSingleton &){} = delete;  // 禁止赋值
}
```

### call_once 实现

```cpp
template <typename T>
class Singleton {
private:
    Singleton();
    ~Singleton();
    Singleton(const Singleton&) = delete;
    Singleton& operator = (const Singleton&) = delete;

private:
    static std::once_flag of;
    static T * instance;
    static void Create() {
        instance = new T;
    }
public:
    static T& GetInstance() {
        std::call_once(&of, &Singleton::Create);
        return *instance;
    }
}

template <typename T>
once_flag Singleton<T>::of;

template <typename T>
T* Singleton<T>::instance = nullptr;
```