# Protocol Buffers

## 序列化与反序列化

- 序列化：将数据结构或对象转换成二进制串的过程

- 反序列化：将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程

## C++ 常用序列化方法

1. xml
2. proto buf
3. boost serialization
4. MFC serialization

## ProtoBuf

### 数据类型

proto 文件消息类型|C++ 类型|说明|
-|-|-|
double|double|双精度浮点|
float|float|单精度浮点|
int32|int32|使用可变长编码方式，负数时不够高效，应该使用sint32|
int64|int64|同上|
uint32|uint32|使用可变长编码方式|
uint64|uint64|同上|
sint32|int32|使用可变长编码方式，有符号的整型值，负数编码时比通常的 int32 高效|
sint64|int64|同上|
fixed32|uint32|总是 4 个字节，如果数值总是比 2^28 大的话，这个类型会比 uint32 高效|
fixed64|uint64|总是 8 个字节，如果数值总是比 2^56 大的话，这个类型会比 uint64 高效|
sfixed32|int32|总是 4 个字节|
sfixed64|int64|总是 8 个字节|
bool|bool||
string|string|一个字符串必须是 utf-8 编码或者 7-bit 的 ascii 编码的文本|
bytes|string|可能包含任意顺序的字节数据|

### 定义 proto 文件

定义 proto 文件就是定义自己的数据存储或者传输的协议格式

为每一个你需要序列化的数据结构添加一个message，然后为消息 message 中的每一个字段 field 指定一个名字、类型和修饰符以及唯一标识 tag

每一个 message 对应到 C++ 中就是一个类，嵌套消息对应的就是嵌套类，当然一个 proto 文件中可以定义多个消息，就像一个头文件中可以定义多个类一样

```
syntax = "proto3";

package tutorial;

message Student {
	uint64 id = 1;
	string name =2;
	string email = 3;
	
	enum PhoneType {
		MOBILE = 0;
		HOME = 1;
	}
	
	message PhoneNumber { 
		string number = 1;
	    PhoneType type = 2;
	}
	repeated PhoneNumber phone = 4;
}

/*
package：.proto 文件以一个 package 声明开始，这个声明是为了防止不同项目之间的命名冲突

字段类型：许多标准的、简单的数据类型都可以用作字段类型，也可以使用其他的 message 类型来作为你的字段类型

修饰符：
1. 单数（默认）：该字段可以出现 0 或 1 次（不能大于 1 次）
2. 可重复（repeated）：该字段可以出现任意次（包含 0）。 可重复字段数值的顺序是系统预定义的，在其后用特殊选项 [packed=true] 来申明以获得更高效的编码

标识："= 1", "= 2" 的标志指出了该字段在二进制编码中使用的唯一标识 tag；标签数字 1-15 的编码比更大的数字少需要一个字节，因此作为一种优化，你可以将这些标签用于经常使用的元素或 repeated 元素，剩下 16 以及更高的标签用于非经常使用的元素或 optional 元素。每一个 repeated 字段的元素需要重新编码标签数字
*/
```

### 编译 proto 文件

```
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR path/to/file.proto

/*
--cpp_out：生成 C++ 类

IMPORT_PATH 指定一个目录用于查找 .proto 文件, 当解析导入命令时. 缺省使用当前目录. 多个导入命令可以通过传递多次

proto_path 选项来指定.这些路径将按照顺序被搜索.-I 等同于 --proto_path， -I=IMPORT_PATH 是 --proto_path 的缩写形式
*/
```

### protobuf API

每个 message 都生成了一个类，为每个字段提供了读写函数

```cpp
// required uint64 id = 1;
  inline bool has_id() const;
  inline void clear_id();
  static const int kIdFieldNumber = 1;
  inline ::google::protobuf::uint64 id() const;
  inline void set_id(::google::protobuf::uint64 value);

  // required string name = 2;
  inline bool has_name() const;
  inline void clear_name();
  static const int kNameFieldNumber = 2;
  inline const ::std::string& name() const;
  inline void set_name(const ::std::string& value);
  inline void set_name(const char* value);
  inline void set_name(const char* value, size_t size);
  inline ::std::string* mutable_name();
  inline ::std::string* release_name();
  inline void set_allocated_name(::std::string* name);

  // optional string email = 3;
  inline bool has_email() const;
  inline void clear_email();
  static const int kEmailFieldNumber = 3;
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline void set_email(const char* value, size_t size);
  inline ::std::string* mutable_email();
  inline ::std::string* release_email();
  inline void set_allocated_email(::std::string* email);

  // repeated .tutorial.Student.PhoneNumber phone = 4;
  inline int phone_size() const;
  inline void clear_phone();
  static const int kPhoneFieldNumber = 4;
  inline const ::tutorial::Student_PhoneNumber& phone(int index) const;
  inline ::tutorial::Student_PhoneNumber* mutable_phone(int index);
  inline ::tutorial::Student_PhoneNumber* add_phone();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Student_PhoneNumber >& phone() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Student_PhoneNumber >* mutable_phone();
```

getter 函数具有与字段名一模一样的名字，并且是小写的，而 setter 函数都是以 set_ 前缀开头

has_ 前缀的函数，对每一个单一的（required 或 optional 的）字段来说，如果字段被置（set）了值，该函数会返回true

每一个字段还有一个 clear_ 前缀的函数，用来将字段重置（un-set）到空状态（empty state）

string 类型字段还有前缀为 mutable_ 的函数返回 string 的直接指针

重复的字段也有一些特殊的函数，得到重复字段的 _size，通过索引（index）来获取一个指定的电话号码，通过指定的索引（index）来更新一个已经存在的电话号码，向消息（message）中添加另一个电话号码，然后你可以编辑它（重复的标量类型有一个add_前缀的函数，允许你传新值进去）

每一个消息（message）还包含了其他一系列函数，用来检查或管理整个消息

```cpp
bool IsInitialized() const; //检查是否全部的required字段都被置（set）了值。

void CopyFrom(const Person& from); //用外部消息的值，覆写调用者消息内部的值。

void Clear();	//将所有项复位到空状态（empty state）。

int ByteSize() const;	//消息字节大小
```

Debug API

```cpp
string DebugString() const;	//将消息内容以可读的方式输出

string ShortDebugString() const; //功能类似于，DebugString(),输出时会有较少的空白

string Utf8DebugString() const; //Like DebugString(), but do not escape UTF-8 byte sequences.

void PrintDebugString() const;	//Convenience function useful in GDB. Prints DebugString() to stdout.
```

解析 API

```cpp
bool SerializeToString(string* output) const; //将消息序列化并储存在指定的string中。注意里面的内容是二进制的，而不是文本；我们只是使用string作为一个很方便的容器。

bool ParseFromString(const string& data); //从给定的string解析消息。

bool SerializeToArray(void * data, int size) const	//将消息序列化至数组

bool ParseFromArray(const void * data, int size)	//从数组解析消息

bool SerializeToOstream(ostream* output) const; //将消息写入到给定的C++ ostream中。

bool ParseFromIstream(istream* input); //从给定的C++ istream解析消息。
```