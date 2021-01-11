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

提供了一些写入日志的方法

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
    void staticCheck() {
        /* numeric_limits<class T> 
        提供了一些获取数学类型数据信息的静态方法，eg：
        std::numeric_limits<T>::is_signed 
        std::numeric_limits<T>::is_integer
        std::numeric_limits<T>::digits
        std::numeric_limits<T>::digits10
        std::numeric_limits<T>::min
        */
        static_assert(kMaxNumericSize - 10 > std::numeric_limits<double>::digits10,
                "kMaxNumericSize is large enough");
        static_assert(kMaxNumericSize - 10 > std::numeric_limits<long double>::digits10,
                "kMaxNumericSize is large enough");
        static_assert(kMaxNumericSize - 10 > std::numeric_limits<long>::digits10,
                "kMaxNumericSize is large enough");
        static_assert(kMaxNumericSize - 10 > std::numeric_limits<long long>::digits10,
                "kMaxNumericSize is large enough");
    }

    template<typename T>
    void formatInteger(T);

    Buffer buffer_;

    // 当 buffer 中剩余空间不小于 kMaxNumericSize 时才允许写入
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
// .h 文件声明一个全局变量
extern Logger::LogLevel g_logLovel;
// .cpp 文件定义
Logger::LogLevel initLogLevel() {
    if (::getenv("MUDUO_LOG_TRACE"))
        return Logger::TRACE;
    else if (::getenv("MUDUO_LOG_DEBUG"))
        return Logger::DEBUG;
    else
        return Logger::INFO;
}
// g_logLevel 只会取 0 1 2
Logger::LogLevel g_logLevel = initLogLevel();


class Logger {
private:
    class Impl {
    public:
        typedef Logger::LogLevel LogLevel;
        Impl(LogLevel level, int old_errno, 
            const SourceFile& file, int line)
                : time_(Timestamp::now())
                , stream_()
                , level_(level)
                , basename_(file) {
            formatTime();
            CurrentThread::tid();
            stream_ << T(CurrentThread::tidString(), CurrentThread::tidStringLength());
            stream_ << T(LogLevelName[level], 6);
            if (savedErrno != 0) {
                stream_ << strerror_tl(savedErrno) << " (errno=" << savedErrno << ") ";
            }
        }
        void formatTime();
        void finish() {
            stream_ << " - " << basename_ << ':' << line_ << '\n';
        }

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

    LogStream& stream() { return impl_.stream_; }

    static LogLevel logLevel() {
        return g_logLevel;
    }
    static void setLogLevel(LogLevel level) {
        g_logLevel = level;
    }

    typedef void (*OutputFunc)(const char* msg, int len);
    typedef void (*FlushFunc)();
    static void setOutput(OutputFunc out) {
        g_output = out;
    }
    static void setFlush(FlushFunc flush) {
        g_flush = flush;
    }

    ~Logger() {
        impl_.finish();
        const LogStream::Buffer& buf(stream().buffer());
        g_output(buf.data(), buf.length());
        if (impl_.level_ == FATAL) {
            g_flush();
            abort();
        }
    }
}

// 由 g_output 来定义日志的输出地
void defaultOutput(const char* msg, int len) {
    size_t n = fwrite(msg, 1, len, stdout);
    //FIXME check n
    (void)n;
}

void defaultFlush() {
    fflush(stdout);
}

Logger::OutputFunc g_output = defaultOutput;
Logger::FlushFunc g_flush = defaultFlush;

// 当定义的全局日志级别小于等于指定日志级别时才打印
// __FILE__、__LINE__、__func__ 是编译器内置宏
// __FILE__ 当前文件名
// __LINE__ 当前行号
// __func__ 当前函数名
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

## LogFile

输出日志文件

```cpp
class LogFile : noncopyable {
private：
    const string basename_;
    const off_t rollSize_;  // 单个日志文件最大值
    const int flushInterval_;
    
    // 当写入多条日志后进行检查是否需要使用新的日志文件
    const int checkEveryN_; 

    // 当前日志文件已写入日志量
    int count_;

    std::unique_ptr<MutexLock> mutex_;
    
    // 更新日志文件相关参数
    time_t startOfPeriod_;
    time_t lastRoll_;
    time_t lastFlush_;

    std::unique_ptr<FileUtil::AppendFile> file_;

    const static int kRollPerSeconds_ = 60*60*24;

public:
    LogFile(const string& basename,
            off_t rollSize,
            bool threadSafe = true,
            int flushInterval = 3,
            int checkEveryN = 1024)
        : basename_(basename),
          rollSize_(rollSize),
          flushInterval_(flushInterval),
          checkEveryN_(checkEveryN),
          count_(0),
          mutex_(threadSafe ? new MutexLock : NULL),
          startOfPeriod_(0),
          lastRoll_(0),
          lastFlush_(0) {
        assert(basename.find('/') == string::npos);
        rollFile();
    }

    ~LogFile() = default;

    void append(const char* logline, int len) {
        if (mutex_) {
            MutexLockGuard lock(*mutex_);
            append_unlocked(logline, len);
        }
        else {
            append_unlocked(logline, len);
        }
    }

    void flush() {
        if (mutex_) {
            MutexLockGuard lock(*mutex_);
            file_->flush();
        }
        else {
            file_->flush();
        }
    }

    // 创建日志文件
    bool rollFile() {
        time_t now = 0;
        string filename = getLogFileName(basename_, &now);
        time_t start = now / kRollPerSeconds_ * kRollPerSeconds_;

        if (now > lastRoll_) {
            lastRoll_ = now;
            lastFlush_ = now;
            startOfPeriod_ = start;
            file_.reset(new FileUtil::AppendFile(filename));
            return true;
        }
        return false;
    }

 private:
    void append_unlocked(const char* logline, int len) {
        file_->append(logline, len);
        if (file_->writtenBytes() > rollSize_) {
            // 当一个日志文件写入足够多的信息后
            // 将重新新建日志文件，保证不全写在一个日志文件
            rollFile();
        }
        else {
            ++count_;
            if (count_ >= checkEveryN_) {
                count_ = 0;
                time_t now = ::time(NULL);
                time_t thisPeriod_ = now / kRollPerSeconds_ * kRollPerSeconds_;
                if (thisPeriod_ != startOfPeriod_) {
                    rollFile();
                }
                else if (now - lastFlush_ > flushInterval_)
                {
                    lastFlush_ = now;
                    file_->flush();
                }
            }
        }
    }

    static string getLogFileName(const string& basename, 
                    time_t* now) {
        string filename;
        filename.reserve(basename.size() + 64);
        filename = basename;

        char timebuf[32];
        struct tm tm;
        *now = time(NULL);
        gmtime_r(now, &tm); // FIXME: localtime_r ?
        strftime(timebuf, sizeof timebuf, ".%Y%m%d-%H%M%S.", &tm);
        filename += timebuf;

        filename += ProcessInfo::hostname();

        char pidbuf[32];
        snprintf(pidbuf, sizeof pidbuf, 
                ".%d", ProcessInfo::pid());
        filename += pidbuf;

        filename += ".log";

        return filename;
    }

}
```

## AsyncLogging

```cpp
class AsyncLogging : noncopyable {
private:
    typedef muduo::detail::FixedBuffer<muduo::detail::kLargeBuffer> Buffer;
    typedef std::vector<std::unique_ptr<Buffer>> BufferVector;
    typedef BufferVector::value_type BufferPtr;

    void threadFunc();

    const int flushInterval_;
    std::atomic<bool> running_;
    const string basename_;
    const off_t rollSize_;
    muduo::Thread thread_;
    muduo::CountDownLatch latch_;
    muduo::MutexLock mutex_;
    muduo::Condition cond_ GUARDED_BY(mutex_);
    BufferPtr currentBuffer_ GUARDED_BY(mutex_);
    BufferPtr nextBuffer_ GUARDED_BY(mutex_);
    BufferVector buffers_ GUARDED_BY(mutex_);

public:
    AsyncLogging(const string& basename,
            off_t rollSize,
            int flushInterval = 3) 
            : flushInterval_(flushInterval)
            , running_(false)
            , basename_(basename)
            , rollSize_(rollSize)
            , thread_(std::bind(&AsyncLogging::threadFunc, this), "Logging")
            , latch_(1)
            , mutex_()
            , cond_(mutex_)
            , currentBuffer_(new Buffer)
            , nextBuffer_(new Buffer)
            , buffers_() {
        currentBuffer_->bzero();
        nextBuffer_->bzero();
        buffers_.reserve(16);
    }

    ~AsyncLogging() {
        if (running_) {
            stop();
        }
    }

    void append(const char* logline, int len);

    void start() {
        running_ = true;
        thread_.start();
        latch_.wait();
    }

    void stop() NO_THREAD_SAFETY_ANALYSIS {
        running_ = false;
        cond_.notify();
        thread_.join();
    }
}

void AsyncLogging::append(const char* logline, int len) {
    muduo::MutexLockGuard lock(mutex_);
    if (currentBuffer_->avail() > len) {
        currentBuffer_->append(logline, len);
    }
    else {
        // 移动拷贝
        buffers_.push_back(std::move(currentBuffer_));

        if (nextBuffer_) {
            currentBuffer_ = std::move(nextBuffer_);
        }
        else {
            currentBuffer_.reset(new Buffer); // Rarely happens
        }
        currentBuffer_->append(logline, len);
        cond_.notify();
    }
}

void AsyncLogging::threadFunc() {
    assert(running_ == true);
    latch_.countDown();
    LogFile output(basename_, rollSize_, false);
    BufferPtr newBuffer1(new Buffer);
    BufferPtr newBuffer2(new Buffer);
    newBuffer1->bzero();
    newBuffer2->bzero();
    BufferVector buffersToWrite;
    buffersToWrite.reserve(16);
    while (running_) {
        assert(newBuffer1 && newBuffer1->length() == 0);
        assert(newBuffer2 && newBuffer2->length() == 0);
        assert(buffersToWrite.empty());

        // 缩短加锁代码段
        {
            muduo::MutexLockGuard lock(mutex_);
            if (buffers_.empty()) {
                cond_.waitForSeconds(flushInterval_);
            }
            buffers_.push_back(std::move(currentBuffer_));
            currentBuffer_ = std::move(newBuffer1);
            buffersToWrite.swap(buffers_);
            if (!nextBuffer_) {
                nextBuffer_ = std::move(newBuffer2);
            }
        }

        assert(!buffersToWrite.empty());

        if (buffersToWrite.size() > 25) {
            char buf[256];
            snprintf(buf, sizeof buf, "Dropped log messages at %s, %zd larger buffers\n",
                    Timestamp::now().toFormattedString().c_str(),
                    buffersToWrite.size()-2);
            fputs(buf, stderr);
            output.append(buf, static_cast<int>(strlen(buf)));
            buffersToWrite.erase(buffersToWrite.begin()+2, buffersToWrite.end());
        }

        for (const auto& buffer : buffersToWrite) {
            output.append(buffer->data(), buffer->length());
        }

        if (buffersToWrite.size() > 2) {
            buffersToWrite.resize(2);
        }

        if (!newBuffer1) {
            assert(!buffersToWrite.empty());
            newBuffer1 = std::move(buffersToWrite.back());
            buffersToWrite.pop_back();
            newBuffer1->reset();
        }

        if (!newBuffer2) {
            assert(!buffersToWrite.empty());
            newBuffer2 = std::move(buffersToWrite.back());
            buffersToWrite.pop_back();
            newBuffer2->reset();
        }

        buffersToWrite.clear();
        output.flush();
    }
    output.flush();
}
```