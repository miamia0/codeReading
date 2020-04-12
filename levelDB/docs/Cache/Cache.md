## Cache

### 模块概要

`include/leveldb/cache.h ` 

`util/cache.cc`

使用LRU方法实现对cache的控制

### 模块实现

| Cache          |                                                              |
| -------------- | ------------------------------------------------------------ |
| Cache 构造函数 |                                                              |
| ~Cache析构函数 |                                                              |
| Insert         | 插入key value。并且传入deleter指针，如果不在需要这个entry时使用deleter删除这个entry。返回handle |
| Lookup         | 对于一个key返回对应的handle                                  |
| Release        | 通过调用release(handle)释放key-value 的cache空间             |
| Value          | 返回handle的值。                                             |
| Erase          | entry将保留到释放所有现有的handle之前。                      |
| NewId          | 返回一个新的数字id。可能会被共享cache的多个client调用        |
|                |                                                              |

#### LRUHandle

lruhandle是一个双向链表。存储在HandleTable（hash）里。

```c++
struct LRUHandle {
  void* value;
  void (*deleter)(const Slice&, void* value);
  LRUHandle* next_hash;
  LRUHandle* next;
  LRUHandle* prev;
  size_t charge;  // TODO(opt): Only allow uint32_t?
  size_t key_length;
  bool in_cache;     // Whether entry is in the cache.
  uint32_t refs;     // References, including cache reference, if present.
  uint32_t hash;     // Hash of key(); used for fast sharding and comparisons
  char key_data[1];  // Beginning of key

  Slice key() const {
    // next_ is only equal to this if the LRU handle is the list head of an
    // empty list. List heads never have meaningful keys.
    assert(next != this);

    return Slice(key_data, key_length);
  }
};
```

`key_data[1]`成员变量这里虽然只设置了长度为1。但是在新建一个handle时会使用以下代码，所以申请的空间刚好是字符串的大小，不需要通过char*的指针二次读取。

```c++
  LRUHandle* e =
      reinterpret_cast<LRUHandle*>(malloc(sizeof(LRUHandle) - 1 + key.size()));
```

#### HandleTable

handleTable是handle的哈希表。使用链表法解决冲突。

具体方式是，开辟`lenghth_` = 2^k大小的数组。对于一个`key`的`hash`值，取`hash&(length_-1)`作为数组中的位置。如果当前table里的handle个数多于`length_`，将`length_`大小扩充为原来的两倍并进行转移。

#### LRUCache

Insert  ,Insert  ,Release  ,Erase,Prune, 和chache的方法基本是相同的，只是增加了key，hash。

```c++
  size_t capacity_;

  // mutex_ protects the following state.
  mutable port::Mutex mutex_;//链表的并发控制
  size_t usage_ ;

  LRUHandle lru_ ;//虚假的Handle list，prev存储最新的entry，next存储最旧的entry。所有的LRU entry都是ref == 1，in_cache == true

  LRUHandle in_use_ ;//虚假的Handle，prev存储最新的entry，next存储最旧的entry。所有的LRU entry都是efs >= 2 and in_cache==true.

  HandleTable table_;//当前在使用的handle的hash表
```

LRUCache使用LRU的方法对cache进行管理。

#### ShardedLRUCache

ShardedLRUCache继承于cache，管理16个LRUCache。

key通过hash值分的shard，因为hash是32位的hash值，所以取最高4位的进行分shard。

```c++
  static uint32_t Shard(uint32_t hash) { return hash >> (32 - kNumShardBits); }
```

所有操作Insert  ,Insert  ,Release  ,Erase,Prune都是调用LRUCache中的操作。

### 相关语法:

1. [匿名空间使用](https://www.cnblogs.com/youxin/p/4308364.html)

2. //TODO(mutable)

3. //TODO(THREAD_ANNOTATION_ATTRIBUTE__)
4. [reinterpret_cast](https://www.cnblogs.com/ider/archive/2011/07/30/cpp_cast_operator_part3.html])
5. [override关键字](https://blog.csdn.net/a435262767/article/details/90898941?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-11&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-11)

### reference 

http://luodw.cc/2015/10/24/leveldb-11/