# define

用一个标识符来表示一个字符串，称为宏。在编译预处理时，对程序中所有出现的宏名，都用宏定义中的字符串去代换。预处理程序对它不作任何检查。如有错误，只能在编译已被宏展开后的源程序时发现

宏定义不是说明或语句，在行末不必加分号，如加上分号则连分号也一起置换

## 无参宏

简单的文本替换

```cpp
#define INT1 int *
typedef int * INT2;

INT1 a1, b1;
INT2 a2, b2;
// INT1 a1, b1 被替换为 int *a1, b1
// 即一个指向int类型的指针一个int类型的变量
// INT2 a2,b2 则是两个指向 int 类型的指针
```

## 有参宏

宏名中不能有空格，宏名与形参表之间也不能有空格，而形参表中形参之间可以有空格

```cpp
#define COUNT(M) M*M
int x=6;
print(COUNT(x+1));  // 13
print(COUNT(++X));  // 56
```

宏定义也可以定义表达式或多个语句

```cpp
#define AB(a,b) a=i+5;b=j+3;   宏定义多个语句
```

## 符号

`#` 的作用就是将 `#` 后边的宏参数进行字符串的操作，也就是将 `#` 后边的参数两边加上一对双引号使其成为字符串

```cpp
#define TEST(param) #param

char *pStr=TEST(123);
printf("pSrt=%s\n",pStr);  // 123
```

`##` 运算符也可以用在替换文本中，它的作用起到粘合的作用，即将两个宏参数连接为一个数

```cpp
#define TEST(param1,param2) (param1##param2)
int num =TEST(12, 34);  // 1234
char* str = Conn("asdf", "adf"); // asdfadf
```

`#@` 字符化操作，将传的单字符参数名转换成字符，以一对单引用括起来，结果返回是一个 `const char`

```cpp
#define ToChar(x)     #@x
char a = ToChar(1);  // '1'
char b = ToChar(123);  // '3'
```

`\` 行继续操作，表示下一行继续此宏的定义，最后一行不要加续行符

```cpp
#define MACRO(arg1,arg2) do { \
    stmt1; \
    stmt2; \
} while(0) /* (no trailing ; ) */

// 利用宏来批量定义
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

#define HTTP_STATUS_MAP(XX)                                                 \
  XX(100, CONTINUE,                        Continue)                        \
  XX(101, SWITCHING_PROTOCOLS,             Switching Protocols)             \
  XX(102, PROCESSING,                      Processing)                      \
  XX(200, OK,                              OK)                              \
  XX(201, CREATED,                         Created)                         \
  XX(202, ACCEPTED,                        Accepted)                        \
  XX(203, NON_AUTHORITATIVE_INFORMATION,   Non-Authoritative Information)   \
  XX(204, NO_CONTENT,                      No Content)                      \
  XX(205, RESET_CONTENT,                   Reset Content)                   \
  XX(206, PARTIAL_CONTENT,                 Partial Content)                 \
  XX(207, MULTI_STATUS,                    Multi-Status)                    \
  XX(208, ALREADY_REPORTED,                Already Reported)                \
  XX(226, IM_USED,                         IM Used)                         \
  XX(300, MULTIPLE_CHOICES,                Multiple Choices)                \
  XX(301, MOVED_PERMANENTLY,               Moved Permanently)               \
  XX(302, FOUND,                           Found)                           \
  XX(303, SEE_OTHER,                       See Other)                       \
  XX(304, NOT_MODIFIED,                    Not Modified)                    \
  XX(305, USE_PROXY,                       Use Proxy)                       \
  XX(307, TEMPORARY_REDIRECT,              Temporary Redirect)              \
  XX(308, PERMANENT_REDIRECT,              Permanent Redirect)              \
  XX(400, BAD_REQUEST,                     Bad Request)                     \
  XX(401, UNAUTHORIZED,                    Unauthorized)                    \
  XX(402, PAYMENT_REQUIRED,                Payment Required)                \
  XX(403, FORBIDDEN,                       Forbidden)                       \
  XX(404, NOT_FOUND,                       Not Found)                       \
  XX(405, METHOD_NOT_ALLOWED,              Method Not Allowed)              \
  XX(406, NOT_ACCEPTABLE,                  Not Acceptable)                  \
  XX(407, PROXY_AUTHENTICATION_REQUIRED,   Proxy Authentication Required)   \
  XX(408, REQUEST_TIMEOUT,                 Request Timeout)                 \
  XX(409, CONFLICT,                        Conflict)                        \
  XX(410, GONE,                            Gone)                            \
  XX(411, LENGTH_REQUIRED,                 Length Required)                 \
  XX(412, PRECONDITION_FAILED,             Precondition Failed)             \
  XX(413, PAYLOAD_TOO_LARGE,               Payload Too Large)               \
  XX(414, URI_TOO_LONG,                    URI Too Long)                    \
  XX(415, UNSUPPORTED_MEDIA_TYPE,          Unsupported Media Type)          \
  XX(416, RANGE_NOT_SATISFIABLE,           Range Not Satisfiable)           \
  XX(417, EXPECTATION_FAILED,              Expectation Failed)              \
  XX(421, MISDIRECTED_REQUEST,             Misdirected Request)             \
  XX(422, UNPROCESSABLE_ENTITY,            Unprocessable Entity)            \
  XX(423, LOCKED,                          Locked)                          \
  XX(424, FAILED_DEPENDENCY,               Failed Dependency)               \
  XX(426, UPGRADE_REQUIRED,                Upgrade Required)                \
  XX(428, PRECONDITION_REQUIRED,           Precondition Required)           \
  XX(429, TOO_MANY_REQUESTS,               Too Many Requests)               \
  XX(431, REQUEST_HEADER_FIELDS_TOO_LARGE, Request Header Fields Too Large) \
  XX(451, UNAVAILABLE_FOR_LEGAL_REASONS,   Unavailable For Legal Reasons)   \
  XX(500, INTERNAL_SERVER_ERROR,           Internal Server Error)           \
  XX(501, NOT_IMPLEMENTED,                 Not Implemented)                 \
  XX(502, BAD_GATEWAY,                     Bad Gateway)                     \
  XX(503, SERVICE_UNAVAILABLE,             Service Unavailable)             \
  XX(504, GATEWAY_TIMEOUT,                 Gateway Timeout)                 \
  XX(505, HTTP_VERSION_NOT_SUPPORTED,      HTTP Version Not Supported)      \
  XX(506, VARIANT_ALSO_NEGOTIATES,         Variant Also Negotiates)         \
  XX(507, INSUFFICIENT_STORAGE,            Insufficient Storage)            \
  XX(508, LOOP_DETECTED,                   Loop Detected)                   \
  XX(510, NOT_EXTENDED,                    Not Extended)                    \
  XX(511, NETWORK_AUTHENTICATION_REQUIRED, Network Authentication Required) \

enum http_status {
#define XX(num, name, string) HTTP_STATUS_##name = num,
	HTTP_STATUS_MAP(XX)
#undef XX
};
```