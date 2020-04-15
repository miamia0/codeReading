# Iterator 

### 模块概要

`include/leveldb/iterator.h`

 `table/iterator.cc`



### 模块接口

```c++
  virtual bool Valid() const = 0;//iter有效性
  virtual void SeekToFirst() = 0;//iterator移动到首位
  virtual void SeekToLast() = 0;//iterator移动到末尾
  virtual void Seek(const Slice& target) = 0;//查找
  virtual void Next() = 0;
  virtual void Prev() = 0;
  virtual Slice key() const = 0;//返回当前的key
  virtual Slice value() const = 0;//返回当前的value
  virtual Status status() const = 0;
  void RegisterCleanup(CleanupFunction function, void* arg1, void* arg2);//注册销毁函数
```



### 模块实现

#### 回调函数

```c++
  struct CleanupNode {
    // True if the node is not used. Only head nodes might be unused.
    bool IsEmpty() const { return function == nullptr; }
    // Invokes the cleanup function.
    void Run() {
      assert(function != nullptr);
      (*function)(arg1, arg2);
    }

    // The head node is used if the function pointer is not null.
    CleanupFunction function;
    void* arg1;
    void* arg2;
    CleanupNode* next;
  };
  CleanupNode cleanup_head_;
```

cleanup_head_是一个链表在洗狗iterator时会调用执行。

```c++
Iterator::~Iterator() {
  if (!cleanup_head_.IsEmpty()) {
    cleanup_head_.Run();
    for (CleanupNode* node = cleanup_head_.next; node != nullptr;) {
      node->Run();
      CleanupNode* next_node = node->next;
      delete node;
      node = next_node;
    }
  }
}
```



### 对应的Iterator//TODO

存储结构（memtable/sstable/block） 都提供对应的 Iterator，另外还有为操作方便封装的特殊 Iterator。

- [`MemTableIterator`](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable) (`db/memtable.cc`)：memtable 对 key 的查找和遍历封装成 `MemTableIterator`。 底层直接使用 SkipList 的类 Iterator 接口
- [`TwoLevelIterator`](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable) (`table/two_level_iterator.cc`)：SSTable 对于类似 index ==> data 这种需要定位 index，然后根据 index 定位到具体 data 的使用方式
- [`Block::Iter`](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable) (`table/block.cc`)：上层对 Block 进行 key 的查找和遍历，封装成 `Block::Iter` 处理
- [`LevelFileNumIterator`](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable) (非 level-0 的 sstable 元信息集合的 Iterator) (`db/version_set.cc`)：level-0 中的 sstable 可能存在 overlap，处理时每个 sstable 单独处理即可
- [`IteratorWrapper`](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable) (`table/iterator_wrapper.h`)：提供了稍作优化的 Iterator 包装， 它会保存每次 `Key() / Valid()`的值， 从而避免每次调用 Iterator 接口产生的 virtural function 调用。另外，若频繁调用时，直接使用 保存的值，比每次计算能有更好的 cpu cache locality
- [`MergingIterator`](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable) (`table/merge.cc`)：`MergingIterator` 内部包含多个 Iterator 的集合，(`children_`)，每个操作，对 `children_` 中每个 Iterator 做同样操作之后按逻辑取边界的值即可
- [`DBIter`](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable) (`db/db_iter.cc`)：对 db 遍历



### 相关语法



### reference 

[https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable/Iterator%20-%202018-10-01%20-%20rsy.md](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/SSTable/Iterator - 2018-10-01 - rsy.md)