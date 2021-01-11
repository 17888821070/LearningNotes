# FileUtil

## ReadSmallFile

只可读

```cpp
// read small file < 64KB
class ReadSmallFile : noncopyable {
private:
    int fd_;
    int err_;
    char buf_[kBufferSize];

public:
    static const int kBufferSize = 64*1024;

    ReadSmallFile(StringArg filename)
        : fd_(::open(filename.c_str(), O_RDONLY | O_CLOEXEC))
        , err_(0) {
            buf_[0] = '\0';
            if (fd_ < 0) {
                err_ = errno;
            }
    }
    ~ReadSmallFile() {
        if(fd_ >= 0) {
            ::close(fd_);
        }
    }

    template<typename String>
    int readToString(int maxSize,
                   String* content,
                   int64_t* fileSize,
                   int64_t* modifyTime,
                   int64_t* createTime) {
        static_assert(sizeof(off_t) == 8, 
                "_FILE_OFFSET_BITS = 64");
        assert(content != NULL);
        int err = err_;
        if (fd_ >= 0) {
            content->clear();

            if (fileSize) {
                struct stat statbuf;
                if (::fstat(fd_, &statbuf) == 0) {
                    if (S_ISREG(statbuf.st_mode)) {
                        *fileSize = statbuf.st_size;
                        content->reserve(static_cast<int>(std::min(implicit_cast<int64_t>(maxSize),*fileSize)));
                    }
                    else if (S_ISDIR(statbuf.st_mode)) {
                        err = EISDIR;
                    }
                    if (modifyTime) {
                        *modifyTime = statbuf.st_mtime;
                    }
                    if (createTime) {
                        *createTime = statbuf.st_ctime;
                    }
                }
                else {
                    err = errno;
                }
            }
            else {
                err = errno;
            }

            while (content->size() < implicit_cast<size_t>(maxSize)) {
                size_t toRead = std::min(implicit_cast<size_t>(maxSize) - content->size(), sizeof(buf_));
                ssize_t n = ::read(fd_, buf_, toRead);
                if (n > 0) {
                    content->append(buf_, n);
                }
                else {
                    if (n < 0) {
                        err = errno;
                    }
                    break;
                }
            }
        }
        return err;
    }


    int readToBuffer(int* size) {
          int err = err_;
          if (fd_ >= 0) {
            ssize_t n = ::pread(fd_, buf_, sizeof(buf_)-1, 0);
            if (n >= 0) {
                if (size) {
                    *size = static_cast<int>(n);
                }
            buf_[n] = '\0';
            }
            else {
                err = errno;
            }
        }
        return err;
    }

    const char* buffer() const { return buf_; }
}
```

## AppendFile

```cpp
class AppendFile : noncopyable {
private:
    size_t write(const char* logline, size_t len) {
        // 使用非线程安全版本
        // 需要在上层保证西城安全性
        return ::fwrite_unlocked(logline, 1, len, fp_);
    }

    FILE* fp_;
    char buffer_[64*1024];
    off_t writtenBytes_;

public:
    explicit AppendFile(StringArg filename) 
            : fp_(::fopen(filename.c_str(), "ae"))
            , writtenBytes_(0){
        assert(fp_);
        // 设置文件缓存区
        // 将文件的缓存区设置为 buffer_
        ::setbuffer(fp_, buffer_, sizeof buffer_);
    }

    ~AppendFile() {
        ::fclose(fp_);
    }

    void append(const char* logline, size_t len) {
        size_t n = write(logline, len);
        size_t remain = len - n;
        while (remain > 0) {
            size_t x = write(logline + n, remain);
            if (x == 0) {
                int err = ferror(fp_);
                if (err) {
                    fprintf(stderr, "AppendFile::append() failed %s\n", strerror_tl(err));
                }
                break;
            }
            n += x;
            remain = len - n; // remain -= x
        }

        writtenBytes_ += len;
    }

    void flush() {
        ::fflush(fp_);
    }

    ff_t writtenBytes() const { return writtenBytes_; }
}
```