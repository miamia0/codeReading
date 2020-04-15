## TableCache

### 模块概要

`db/table_cache.h`

`db/table_cache.cc`

`TableCache` 缓存 `Table` 对象，每个DB一个。

`TableCache` 的 KV 格式：

- 以 `file_number` 作 key
- 以 `TableAndFile` 对应的预加载索引作为 value。

### 模块接口

```c++
 Iterator* NewIterator(const ReadOptions& options, uint64_t file_number,
                        uint64_t file_size, Table** tableptr = nullptr);//对于指定的file返回一个iterator

//如果在指定未见(file_number)中查找到“K”的一个entry，调用handle_result函数。
Status Get(const ReadOptions& options, uint64_t file_number,
             uint64_t file_size, const Slice& k, void* arg,
             void (*handle_result)(void*, const Slice&, const Slice&));

void Evict(uint64_t file_number);//清除这个file_number的cache
```





### 模块实现

`TableCache`的`Get`方法通过传入`SSTabl`e的`file_number`和`file_size`来调用私有的`FindTable`方法来获取该SSTable对应的`Cache::Handle`句柄（即TableCache的对应`entry`），这里同时完成若未命中`TableCache`则打开该`SSTable`对应的文件并预加载索引信息到内存，以`TableAndFile`的形式保存相应句柄，并插入`TableCache`。最后从cache中读取结果。



#### FindTable实现

```c++
Status TableCache::FindTable(uint64_t file_number, uint64_t file_size,
                             Cache::Handle** handle) {
  Status s;
  char buf[sizeof(file_number)];
  EncodeFixed64(buf, file_number);//file_number压缩
  Slice key(buf, sizeof(buf));
  *handle = cache_->Lookup(key);
  if (*handle == nullptr) {//没有在cache中搜索到
    std::string fname = TableFileName(dbname_, file_number);
    RandomAccessFile* file = nullptr;
    Table* table = nullptr;
    s = env_->NewRandomAccessFile(fname, &file);//使用fname查找file
    if (!s.ok()) {//查找不成功
      std::string old_fname = SSTTableFileName(dbname_, file_number);
      if (env_->NewRandomAccessFile(old_fname, &file).ok()) {
        s = Status::OK();
      }
    }
    if (s.ok()) {
      s = Table::Open(options_, file, file_size, &table);//打开文件
    }

    if (!s.ok()) {//打开文件不成功
      assert(table == nullptr);
      delete file;
      // We do not cache error results so that if the error is transient,
      // or somebody repairs the file, we recover automatically.
    } else {//成功打开文件更新handle 插入cache。
      TableAndFile* tf = new TableAndFile;
      tf->file = file;
      tf->table = table;
      *handle = cache_->Insert(key, tf, 1, &DeleteEntry);
    }
  }
  return s;//查询状态
}
```

#### NewIterator实现//TODO(先读table和option的源码)



### 依赖说明

被class DBImpl private 含有。

### 使用流程

来看TableCache，先简单梳理下到`TableCache`的查找流程，用户提交`key`查询，交由`Status DBImpl::Get()`，获取两种`MemTable`和当前`Version`，依次查询`memtable`、`immutable memtable` ，未找到则在当前`Version`上`Status Version::Get()`，依次从最低level到最高level查询直至查到，在每层确定可能包含该`key`的SSTable文件后，就会在所属`VersionSet`的`table_cache`中继续查询，即调用`Status TableCache::Get()`。

### 相关语法



### reference 

[https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/DB/TableCache%20-%202018-09-30%20-%20rsy.md](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/DB/TableCache - 2018-09-30 - rsy.md)

http://www.pandademo.com/2016/04/two-stage-cache-of-sstable-leveldb-source-dissect-8/



