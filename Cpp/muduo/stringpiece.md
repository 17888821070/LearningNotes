# StringPiece

## StringArg

```cpp
class StringArg {
private:
    const char* str_;

public:
    StringArg(const char* str)
        : str_(str) {
    }

    StringArg(const string& str)
        : str_(str.c_str()) {
    }

    const char* c_str() const { 
        return str_; 
    }
}
```

## StringPiece

```cpp
class StringPiece {
private:
    const char* ptr_;
    int         length_;

public:
// 使用宏定义快速定义多个重载运算符
#define STRINGPIECE_BINARY_PREDICATE(cmp,auxcmp)                             \
    bool operator cmp (const StringPiece& x) const {                         \
        int r = memcmp(ptr_, x.ptr_, length_ < x.length_ ?                   \
        length_ : x.length_);                                                \
        return ((r auxcmp 0) || ((r == 0) && (length_ cmp x.length_)));      \
    }
    STRINGPIECE_BINARY_PREDICATE(<,  <);
    STRINGPIECE_BINARY_PREDICATE(<=, <);
    STRINGPIECE_BINARY_PREDICATE(>=, >);
    STRINGPIECE_BINARY_PREDICATE(>,  >);
#undef STRINGPIECE_BINARY_PREDICATE
}
```