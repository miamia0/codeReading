## Slice

### 模块概要

`include/leveldb/slice.h`

slice是数据库中key的类型。存储一个字符串char*和一个字符串长度len。

### 模块接口

```c++
inline int Slice::compare(const Slice& b)//和b比较字典序。
bool starts_with(const Slice& x)//判断x是否是前缀
void remove_prefix(size_t n) //删除一个前缀。
```

### 模块实现

remove_prefix直接将指针后移并修改`size_`大小，并没有对char的空间做任何修改。这里的Slice中的字符串应该是只读的。空间的释放应该在外部进行。后续读代码的时候可以看一下辣鸡处理的方式。

```c++
    data_ += n;
    size_ -= n;
```

### 相关语法



