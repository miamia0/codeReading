## MemTable

### 模块概要

`db/memtable.cc`

`db/memtable.h`



### 模块接口

```c++
  void Add(SequenceNumber seq, ValueType type, const Slice& key,
           const Slice& value);
  bool Get(const LookupKey& key, std::string* value, Status* s);

  friend class MemTableIterator;            
  friend class MemTableBackwardIterator; //没有找到实现
  Iterator* NewIterator();

  struct KeyComparator {
    const InternalKeyComparator comparator;
    explicit KeyComparator(const InternalKeyComparator& c) : comparator(c) {}
    int operator()(const char* a, const char* b) const;
  };

  KeyComparator comparator_;//skeplist用的节点比较方式
  int refs_;//reference 管理
  Arena arena_; //内存分配管理

  typedef SkipList<const char*, KeyComparator> Table;
  Table table_;//SkipList
```

### 模块实现

memtable内部由skeplist实现存储的数据结构，内存分配由Arena实现。

![](pic/memtable.jpg)

### 相关依赖说明

从 WriteBatch 写入：

- `DBImpl::Write(WriteBatch*)`
- `WriteBatchInternal::InsertInto(const WriteBatch*, MemTable*)`
- `WriteBatch::Iterate(MemTableInserter*)`：**将 k-v 不断迭代地插入 memtable**
- `MemTableInserter::Put()` 和 `MemTableInserter::Delete()` 转发给 `Memtable::Add()`

**Dump**：`DBImpl::WriteLevel0Table()` 在 写 level-0（不一定） 时将 `iter` 传入 `BuildTable()` 中写入文件。

```
DBImpl 代码还没有康，等之后再补一下DBImpl与memtable之间的关系
```

### 相关语法

### reference

[https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/DB/Memtable%20-%202018-10-04%20-%20rsy.md](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/DB/Memtable - 2018-10-04 - rsy.md)