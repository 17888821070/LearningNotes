# Http

## HttpRequest

Http 请求头

```cpp
class HttpRequest : public muduo::copyable {
public:
    enum Method {
        kInvalid, kGet, kPost, kHead, kPut, kDelete
    };
    enum Version {
        kUnknown, kHttp10, kHttp11
    };

private:
    Method method_;
    Version version_;
    string path_;
    string query_;
    Timestamp receiveTime_;
    std::map<string, string> headers_;

public:
    HttpRequest() : method_(kInvalid), version_(kUnknown) {

    }

    bool setMethod(const char* start, const char* end) {
        assert(method_ == kInvalid);
        string m(start, end);
        if (m == "GET") {
            method_ = kGet;
        }
        else if (m == "POST") {
            method_ = kPost;
        }
        else if (m == "HEAD") {
            method_ = kHead;
        }
        else if (m == "PUT") {
            method_ = kPut;
        }
        else if (m == "DELETE") {
            method_ = kDelete;
        }
        else {
            method_ = kInvalid;
        }
        return method_ != kInvalid;
    }

    void setPath(const char* start, const char* end) {
        path_.assign(start, end);
    }

    void setQuery(const char* start, const char* end) {
        query_.assign(start, end);
    }

    const char* methodString() const {
        const char* result = "UNKNOWN";
        switch(method_) {
            case kGet:
                result = "GET";
                break;
            case kPost:
                result = "POST";
                break;
            case kHead:
                result = "HEAD";
                break;
            case kPut:
                result = "PUT";
                break;
            case kDelete:
                result = "DELETE";
                break;
            default:
                break;
        }
        return result;
    }

    // colon 指向第一个空格
    // isspace() 是库函数，判断是否为空格、\t、\r、\n、\v、\f
    void addHeader(const char* start, const char* colon, const char* end) {
        string field(start, colon);
        ++colon;
        while (colon < end && isspace(*colon)) {
          ++colon;
        }
        string value(colon, end);
        while (!value.empty() && isspace(value[value.size()-1])) {
          value.resize(value.size()-1);
        }
        headers_[field] = value;
    }
}

```

## HttpContext

```cpp
class HttpContext : public muduo::copyable {
public:
    enum HttpRequestParseState {
        kExpectRequestLine,
        kExpectHeaders,
        kExpectBody,
        kGotAll,
    };

    HttpContext()
      : state_(kExpectRequestLine) {
    }

    // return false if any error
    bool parseRequest(Buffer* buf, Timestamp receiveTime);

private:
    bool processRequestLine(const char* begin, const char* end);

    HttpRequestParseState state_;
    HttpRequest request_;
}

bool HttpContext::processRequestLine(const char* begin, const char* end) {
    bool succeed = false;
    const char* start = begin;
    const char* space = std::find(start, end, ' ');
    if (space != end && request_.setMethod(start, space)) {
        start = space+1;
        space = std::find(start, end, ' ');
        if (space != end) {
            const char* question = std::find(start, space, '?');
            if (question != space) {
                request_.setPath(start, question);
                request_.setQuery(question, space);
            }
            else {
                request_.setPath(start, space);
            }
            start = space+1;
            succeed = end-start == 8 && std::equal(start, end-1, "HTTP/1.");
            if (succeed) {
                if (*(end-1) == '1') {
                    request_.setVersion(HttpRequest::kHttp11);
                }
                else if (*(end-1) == '0') {
                    request_.setVersion(HttpRequest::kHttp10);
                }
                else {
                    succeed = false;
                }
            }
        }
    }
    return succeed;
}

bool HttpContext::parseRequest(Buffer* buf, Timestamp receiveTime) {
    bool ok = true;
    bool hasMore = true;
    while (hasMore) {
        if (state_ == kExpectRequestLine) {
            const char* crlf = buf->findCRLF();
            if (crlf) {
                ok = processRequestLine(buf->peek(), crlf);
                if (ok) {
                    request_.setReceiveTime(receiveTime);
                    buf->retrieveUntil(crlf + 2);
                    state_ = kExpectHeaders;
                }
                else {
                    hasMore = false;
                }
            }
            else {
                hasMore = false;
            }
        }
        else if (state_ == kExpectHeaders) {
            const char* crlf = buf->findCRLF();
            if (crlf) {
                const char* colon = std::find(buf->peek(), crlf, ':');
                if (colon != crlf) {
                    request_.addHeader(buf->peek(), colon, crlf);
                }
                else {
                // empty line, end of header
                // FIXME:
                    state_ = kGotAll;
                    hasMore = false;
                }
                buf->retrieveUntil(crlf + 2);
            }
            else {
                hasMore = false;
            }
        }
        else if (state_ == kExpectBody) {
            // FIXME:
        }
    }
    return ok;
}
```