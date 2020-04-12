## Status

### 模块概要

`util/status.cc`

`include/leveldb/status.h`

status用来判断leveldb操作的状态，可能是sucess也可能是error message。

线程安全性

### 重要模块接口

```c++
static Status NotFound(const Slice& msg, const Slice& msg2 = Slice());//返回notfoundstatus
static Status Corruption(const Slice& msg, const Slice& msg2 = Slice());//返回corruption status
...
Status(Code code, const Slice& msg, const Slice& msg2);//创建一个新的status
static const char* CopyState(const char* s);//复制s的字符串
```

### 模块实现

#### 存储一个操作的状态

```c++
  enum Code {
    kOk = 0,
    kNotFound = 1,
    kCorruption = 2,
    kNotSupported = 3,
    kInvalidArgument = 4,
    kIOError = 5
  };
  // OK status has a null state_.  Otherwise, state_ is a new[] array
  // of the following form:
  //    state_[0..3] == length of message
  //    state_[4]    == code
  //    state_[5..]  == message
  const char* state_;//state_存储完整的message
```

#### 重载=

```c++
inline Status& Status::operator=(const Status& rhs) {
  if (state_ != rhs.state_) {
    delete[] state_;
    state_ = (rhs.state_ == nullptr) ? nullptr : CopyState(rhs.state_);
  }
  return *this;
}
inline Status& Status::operator=(Status&& rhs) noexcept {
  std::swap(state_, rhs.state_);
  return *this;
}
```

对于=重载了左值和右值，左值选择copy状态，右值选择直接swap。



#### 状态复制

```c++
const char* Status::CopyState(const char* state) {
  uint32_t size;
  memcpy(&size, state, sizeof(size));
  char* result = new char[size + 5];
  memcpy(result, state, size + 5);
  return result;
}
```

一个字节是8bit，uint32_t 是4字节，state的前四字节字符串的大小信息。所以`memcpy(&size, state, sizeof(size));`将前四个字节复制给了size。从而可以得到返回值result的大小。

### 相关语法

1.TODO(关键字noexcept)

2.静态成员函数：只能访问静态成员变量和静态成员函数。

3.[左值引用与右值引用](https://blog.csdn.net/qianyayun19921028/article/details/80875002)





