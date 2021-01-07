# Log

## 转换函数

```cpp
const char digits[] = "9876543210123456789";
const char* zero = digits + 9;  // 指向 0

// T ----> string
template<typename T>
size_t convert(char buf[], T value) {
    T i = value;
    char* p = buf;

    do {
        int lsd = static_cast<int>(i % 10);
        i /= 10;
        *p++ = zero[lsd];
    } while (i != 0);

    if (value < 0) {
        *p++ = '-';
    }
    *p = '\0';
    std::reverse(buf, p);
    return p - buf;
}

size_t convertHex(char buf[], uintptr_t value) {
    uintptr_t i = value;
    char* p = buf;

    do {
        int lsd = static_cast<int>(i % 16);
        i /= 16;
        *p++ = digitsHex[lsd];
    } while (i != 0);

    *p = '\0';
    std::reverse(buf, p);
    return p - buf;
}
```

## FixBuffer

用来存储日志内容

```cpp
template<int SIZE>  // 带类型参数的模板类
class FixdBuffer {
private:
    // 左闭右开
    const char* end() const { return data_ + sizeof data_; }

    static void cookieStart();
    static void cookieEnd();

    void (*cookie_)();  // 函数指针
    char data_[SIZE];
    char* cur_;

public:
    FixedBuffer() : cur_(data_) {
        setCookie(cookieStart);
    }

    ~FixedBuffer() {
        setCookie(cookieEnd);
    }

    void setCookie(void (*cookie)()) { cookie_ = cookie; }

    int avail() const { return static_cast<int>(end() - cur_); }
    int length() const { return static_cast<int>(cur_ - data_); }
    void bzero() { memZero(data_, sizeof data_); }

    void append(const char* /*restrict*/ buf, size_t len) {
        if (implicit_cast<size_t>(avail()) > len) {
            memcpy(cur_, buf, len);
            cur_ += len;
        }
    }

    const char* debugString() {
        *cur_ = '\0';
        return data_;
    }

    string toString() const { return string(data_, length()); }
    StringPiece toStringPiece() const { return StringPiece(data_, length()); }
}

template<int SIZE>
void FixedBuffer<SIZE>::cookieStart() {}

template<int SIZE>
void FixedBuffer<SIZE>::cookieEnd() {}
```

## LogStream

将内容写入 FixedBuffer 中

```cpp
class LogStream {
    typedef LogStream self;
public:
    typedef FixedBuffer<1000> Buffer;

    self& operator<<(bool v) {
        buffer_.append(v ? "1" : "0", 1);
        return *this;
    }

    self& operator<<(short) {
        *this << static_cast<int>(v);
        return *this;
    }

    self& operator<<(unsigned short) {
        *this << static_cast<unsigned int>(v);
        return *this;
    }
    
    self& operator<<(int) {
        formatInteger(v);
        return *this;
    }

    self& operator<<(unsigned int) {
        formatInteger(v);
        return *this;
    }

    self& operator<<(long) {
        formatInteger(v);
        return *this;
    }

    self& operator<<(unsigned long) {
        formatInteger(v);
        return *this;
    }

    self& operator<<(long long) {
        formatInteger(v);
        return *this;
    }

    self& operator<<(unsigned long long) {
        formatInteger(v);
        return *this;
    }

    self& operator<<(const void*) {
        uintptr_t v = reinterpret_cast<uintptr_t>(p);
        if (buffer_.avail() >= kMaxNumericSize) {
            char* buf = buffer_.current();
            buf[0] = '0';
            buf[1] = 'x';
            size_t len = convertHex(buf+2, v);
            buffer_.add(len+2);
        }
        return *this;
    }

    self& operator<<(float v) {
        *this << static_cast<double>(v);
        return *this;
    }

    self& operator<<(double v) {
        if (buffer_.avail() >= kMaxNumericSize) {
            // int snprintf(char *str, size_t size, const char *format, ...)
            // C 库函数，将可变参数 (...) 按照 format 格式化成字符串
            // 并将字符串复制到 str 中，size 为要写入的字符的最大数目
            int len = snprintf(buffer_.current(), kMaxNumericSize, "%.12g", v);
            buffer_.add(len);
        }
        return *this;
    }

    self& operator<<(char v) {
        buffer_.append(&v, 1);
        return *this;
    }

    self& operator<<(const char* str) {
        if (str) {
            buffer_.append(str, strlen(str));
        }
        else {
            buffer_.append("(null)", 6);
        }
        return *this;
    }

    self& operator<<(const unsigned char* str) {
        return operator<<(reinterpret_cast<const char*>(str));
    }

    self& operator<<(const string& v) {
        buffer_.append(v.c_str(), v.size());
        return *this;
    }

    self& operator<<(const StringPiece& v) {
        buffer_.append(v.data(), v.size());
        return *this;
    }

    self& operator<<(const Buffer& v) {
        *this << v.toStringPiece();
        return *this;
    }

    void append(const char* data, int len) { buffer_.append(data, len); }

private:
    void staticCheck();

    template<typename T>
    void formatInteger(T);

    Buffer buffer_;

    static const int kMaxNumericSize = 32;
}

template<typename T>
void LogStream::formatInteger(T v) {
    if (buffer_.avail() >= kMaxNumericSize) {
        // 直接加 T 转成 string 并写入 buf
        size_t len = convert(buffer_.current(), v);
        // 转换时已经写入 buf，但 buf 内部长度没变
        buffer_.add(len);
    }
}
```

## Fmt

将算术类型按给定格式转换，并写入 buf 中

```cpp
class Fmt {
private:
    char buf_[32];
    int length_;

public:
    // 将 val 转成 fmt 格式的字符串，并写入 buf 中
    template<typename T>
    Fmt(const char* fmt, T val) {
        // 判断 val 必须是算术类型变量
        static_assert(std::is_arithmetic<T>::value == true, 
                    "Must be arithmetic type");
        length_ = snprintf(buf_, sizeof buf_, fmt, val);
        assert(static_cast<size_t>(length_) < sizeof buf_);
    }

    const char* data() const { return buf_; }
    int length() const { return length_; }
}

inline LogStream& operator<<(LogStream& s, const Fmt& fmt) {
    s.append(fmt.data(), fmt.length());
    return s;
}
```

## Logger

一次 log 事件

```cpp
class Logger {
private:

    class Impl {
    public:
        typedef Logger::LogLevel LogLevel;
        Impl(LogLevel level, int old_errno, const SourceFile& file, int line);
        void formatTime();
        void finish();

        Timestamp time_;
        LogStream stream_;
        LogLevel level_;
        int line_;
        SourceFile basename_;
    };
    Impl impl_;

public:
    enum LogLevel {
        TRACE,
        DEBUG,
        INFO,
        WARN,
        ERROR,
        FATAL,
        NUM_LOG_LEVELS,
    };

    class SourceFile {
    public:
        template<int N>
        SourceFile(const char (&arr)[N]) : data_(arr), size_(N-1) {
            // arr 是 const char [] 的引用，长度为 N
            const char* slash = strrchr(data_, '/'); // builtin function
            if (slash) {
                data_ = slash + 1;
                size_ -= static_cast<int>(data_ - arr);
            }
        }

        explicit SourceFile(const char* filename) : data_(filename) {
            const char* slash = strrchr(filename, '/');
            if (slash) {
                data_ = slash + 1;
            }
            size_ = static_cast<int>(strlen(data_));
        }

        const char* data_;
        int size_;
    };

    Logger(SourceFile file, int line);
    Logger(SourceFile file, int line, LogLevel level);
    Logger(SourceFile file, int line, LogLevel level, const char* func);
    Logger(SourceFile file, int line, bool toAbort);
}




#define LOG_TRACE if (muduo::Logger::logLevel() <= muduo::Logger::TRACE) \
    muduo::Logger(__FILE__, __LINE__, muduo::Logger::TRACE, __func__).stream()
#define LOG_DEBUG if (muduo::Logger::logLevel() <= muduo::Logger::DEBUG) \
    muduo::Logger(__FILE__, __LINE__, muduo::Logger::DEBUG, __func__).stream()
#define LOG_INFO if (muduo::Logger::logLevel() <= muduo::Logger::INFO) \
    muduo::Logger(__FILE__, __LINE__).stream()
#define LOG_WARN muduo::Logger(__FILE__, __LINE__, muduo::Logger::WARN).stream()
#define LOG_ERROR muduo::Logger(__FILE__, __LINE__, muduo::Logger::ERROR).stream()
#define LOG_FATAL muduo::Logger(__FILE__, __LINE__, muduo::Logger::FATAL).stream()
#define LOG_SYSERR muduo::Logger(__FILE__, __LINE__, false).stream()
#define LOG_SYSFATAL muduo::Logger(__FILE__, __LINE__, true).stream()
```