# 类型转换

```cpp
inline void memZero(void* p, size_t n) {
    memset(p, 0, n);
}

template<typename To, typename From>
inline To implicit_cast(From const &f) {
  return f;
}

template<typename To, typename From>
inline To down_cast(From* f) {  
    // only accept pointers
    // Ensures that To is a sub-type of From *.  This test is here only
    // for compile-time type checking, and has no overhead in an
    // optimized build at run-time, as it will be optimized away
    // completely.
    if (false) {
        implicit_cast<From*, To>(0);
    }

    #if !defined(NDEBUG) && !defined(GOOGLE_PROTOBUF_NO_RTTI)
    assert(f == NULL || dynamic_cast<To>(f) != NULL);  // RTTI: debug mode only!
    #endif
    return static_cast<To>(f);
}
```