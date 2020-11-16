# 资源管理方法RAII

RAII 全称是 “Resource Acquisition is Initialization” ，使用 RAII 技术也可以实现安全、简洁的状态管理

在构造函数中申请分配资源，在析构函数中释放资源

智能指针（`std::shared_ptr` 和 `std::unique_ptr`）即 RAII 最具代表的实现，使用智能指针，可以实现自动的内存管理，再也不需要担心忘记 delete 造成的内存泄漏

## 资源管理

内存只是资源的一种，更加广义的资源像文件、 windows 中句柄等等都可以使用 RAII 来进行资源管理

```cpp
void function()
{
    FILE *f = fopen("test.txt", 'r');
    if (.....)
    {
        fclose(f);
        return;
    }
    else if(.....)
    {
        fclose(f);
        return;
    }

    fclose(f);
    ......
}


// 使用 lambda 表达式和 std::function 相结合的方法
#define SCOPEGUARD_LINENAME_CAT(name, line) name##line
#define SCOPEGUARD_LINENAME(name, line) SCOPEGUARD_LINENAME_CAT(name, line)
#define ON_SCOPE_EXIT(callback) ScopeGuard SCOPEGUARD_LINENAME(EXIT, __LINE__)(callback)

class ScopeGuard
{
public:
    explicit ScopeGuard(std::function<void()> f) : 
        handle_exit_scope_(f){};

    ~ScopeGuard(){ handle_exit_scope_(); }
private:
    std::function<void()> handle_exit_scope_;
};

int main()
{
    {
        A *a = new A();
        ON_SCOPE_EXIT([&] {delete a; });
        ......
    }

    {
        std::ofstream f("test.txt");
        ON_SCOPE_EXIT([&] {f.close(); });
        ......
    }

    system("pause");
    return 0;
}


```




