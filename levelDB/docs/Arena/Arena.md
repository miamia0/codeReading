## Arena

### 模块概要

`util/arena.h`

`util/arena.cc`

用于分配空间，主要是memtable的使用进行skiplist的内存分配，没有GC和RC。

### 模块接口

```c++
  char* Allocate(size_t bytes);//申请大小为bytes的空间并返回指针
  char* AllocateAligned(size_t bytes);//分配内存时扩充一点点申请的空间保证首尾与字对齐，
  char* AllocateFallback(size_t bytes);//当前arena内部剩余的内存不够时，新申请内存
  char* AllocateNewBlock(size_t block_bytes);
  size_t MemoryUsage() const//
```

### 模块实现

```c++
  char* alloc_ptr_;//当前使用大块内存的指针
  size_t alloc_bytes_remaining_;//当前使用大块内存的剩余
  std::vector<char*> blocks_;//AllocateNewBlock申请的大块内存
  std::atomic<size_t> memory_usage_;//统计当前arena一共用了多少内存，线程安全。
```



### 相关语法



### 相关依赖

用于在 `memtable` 中分配空间，在 `memtable` 析构时 `arena` 析构。

### reference

[https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/util/Arena/arena%20-%202018-09-30%20-%20rsy.md](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/util/Arena/arena - 2018-09-30 - rsy.md)



