## Comparator

### 模块概要

`util/comparator.cc`
`include/leveldb/comparator.h`
`db/dbformat.h`
`db/dbformat.cc`

comparator提供对于key的比较的方法，保证线程安全。
`BytewiseComparatorImpl`实现普通slice 比较
`InternalKeyComparator` 实现对于internalkey的比较，包含一个`BytewiseComparatorImpl`。

### 模块接口

```c++
  
virtual int Compare(const Slice& a, const Slice& b) const = 0;//比较a和b的大小
virtual void FindShortestSeparator(
      std::string* start,
      const Slice& limit) const = 0;//寻找start和limit的公共前缀并将第一个不同的位置start[diff_index]++,并切分start的大小到diff_index+1，清除之后的数据。(实际上是将start变成逻辑上更大但是物理上更短的字符串)
  //作用类似，但无上端限制(可以认为limit为[0xff,0xff,0xff....]) 
virtual void FindShortSuccessor(std::string* key) const = 0;
```



### 模块实现

#### InternalKey

对于一个internalkey 最后八位存储的是sequence和type的encodefix64，前面存储的是user_key.

```c++

inline bool ParseInternalKey(const Slice& internal_key,
                             ParsedInternalKey* result) {
  const size_t n = internal_key.size();
  if (n < 8) return false;
  uint64_t num = DecodeFixed64(internal_key.data() + n - 8);
  uint8_t c = num & 0xff;
  result->sequence = num >> 8;
  result->type = static_cast<ValueType>(c);
  result->user_key = Slice(internal_key.data(), n - 8);
  return (c <= static_cast<uint8_t>(kTypeValue));
}
```

`InternalKeyComparator`中的`FindShortestSeparator`与`FindShortSuccessor`之后的`number`和`type`用`kMaxSequenceNumber`, `kValueTypeForSeek`代替。

#### LookupKey

lookupKey为DBImpl::Get()提供服务。

```c++
start_                   kstart_                            end_
[array_size(encoded)    ,user_key,  squence_type(encoded)    ]
const char* start_;//开辟空间的初始位置
const char* kstart_;//
const char* end_;
```



### 依赖说明



### 相关语法



### reference 

https://www.cnblogs.com/KevinT/p/3814021.html