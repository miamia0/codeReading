## Coding

### 模块概要

`util/coding.h`
`util/coding.cc`

节约存储空间，将整型编码转换成变长。


### 模块接口
```c++
// Standard Put... routines append to a string
void PutFixed32(std::string* dst, uint32_t value);//写入定长uint32,存在dst中
void PutFixed64(std::string* dst, uint64_t value);
void PutVarint32(std::string* dst, uint32_t value);//写入变长uint32，存在dst中
void PutVarint64(std::string* dst, uint64_t value);
void PutLengthPrefixedSlice(std::string* dst, const Slice& value);//TODO

// Standard Get... routines parse a value from the beginning of a Slice
// and advance the slice past the parsed value.
bool GetVarint32(Slice* input, uint32_t* value);//读取变长uint32并删除这个数字。
bool GetVarint64(Slice* input, uint64_t* value);
bool GetLengthPrefixedSlice(Slice* input, Slice* result);//TODO

// Pointer-based variants of GetVarint...  These either store a value
// in *v and return a pointer just past the parsed value, or return
// nullptr on error.  These routines only look at bytes in the range
// [p..limit-1]
const char* GetVarint32Ptr(const char* p, const char* limit, uint32_t* v);// 将[p,limit)部分的内存解析为变长uint32放到v里面,返回下一个字节指针
const char* GetVarint64Ptr(const char* p, const char* limit, uint64_t* v);//将[p,limit)部分的内存解析为变长uint64放到v里面,返回下一个字节指针

// Returns the length of the varint32 or varint64 encoding of "v"
int VarintLength(uint64_t v);//变长uint32/uint64长度

// Lower-level versions of Put... that write directly into a character buffer
// and return a pointer just past the last byte written.
// REQUIRES: dst has enough space for the value being written
char* EncodeVarint32(char* dst, uint32_t value);//保证小端序
char* EncodeVarint64(char* dst, uint64_t value);

inline uint32_t DecodeFixed32(const char* ptr);
inline uint64_t DecodeFixed64(const char* ptr);
const char* GetVarint32PtrFallback(const char* p, const char* limit,
                                   uint32_t* value);
inline const char* GetVarint32Ptr(const char* p, const char* limit,
                                  uint32_t* value);
```

### 模块实现

* `Encode`以为encodeVarint32为例，1字节8bit，将数据拆分成7bit一组，字节最高为如果为1代表不是最大组，如果为0代表是新开始的数据。采用小端序。

```c++
char* EncodeVarint32(char* dst, uint32_t v) {
  // Operate on characters as unsigneds
  uint8_t* ptr = reinterpret_cast<uint8_t*>(dst);
  static const int B = 128;
  if (v < (1 << 7)) {
    *(ptr++) = v;
  } else if (v < (1 << 14)) {
    *(ptr++) = v | B;
    *(ptr++) = v >> 7;
  } else if (v < (1 << 21)) {
    *(ptr++) = v | B;
    *(ptr++) = (v >> 7) | B;
    *(ptr++) = v >> 14;
  } else if (v < (1 << 28)) {
    *(ptr++) = v | B;
    *(ptr++) = (v >> 7) | B;
    *(ptr++) = (v >> 14) | B;
    *(ptr++) = v >> 21;
  } else {
    *(ptr++) = v | B;
    *(ptr++) = (v >> 7) | B;
    *(ptr++) = (v >> 14) | B;
    *(ptr++) = (v >> 21) | B;
    *(ptr++) = v >> 28;
  }
  return reinterpret_cast<char*>(ptr);
}
```

* `GetVarint32Ptr` 用于 Varint32 解码，该函数处理 只有1byte的情况，否则转发给 `GetVarint32PtrFallback`。

```c++
inline const char* GetVarint32Ptr(const char* p, const char* limit,
                                  uint32_t* value) {
  if (p < limit) {
    uint32_t result = *(reinterpret_cast<const uint8_t*>(p));
    if ((result & 128) == 0) {//字节最高位为0代表单字符。
      *value = result;
      return p + 1;
    }
  }
  return GetVarint32PtrFallback(p, limit, value);
}
const char* GetVarint32PtrFallback(const char* p, const char* limit,
                                   uint32_t* value) {
  uint32_t result = 0;
  for (uint32_t shift = 0; shift <= 28 && p < limit; shift += 7) {
    uint32_t byte = *(reinterpret_cast<const uint8_t*>(p));
    p++;
    if (byte & 128) {
      // More bytes are present
      result |= ((byte & 127) << shift);
    } else {
      result |= (byte << shift);
      *value = result;
      return reinterpret_cast<const char*>(p);
    }
  }
  return nullptr;
}
```



### 相关语法

这里将`char*`类型的p强制转换成` const uint8_t* `指针类型并取指针内容。

```c++
    uint32_t byte = *(reinterpret_cast<const uint8_t*>(p));
```



### reference

* http://mingxinglai.com/cn/2013/01/leveldb-varint32/
* [https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/util/Coding/coding%20-%202018-09-06%20-%20rsy.md](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/util/Coding/coding - 2018-09-06 - rsy.md)