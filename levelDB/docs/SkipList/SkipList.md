## SkipList

### 模块概要

`db/SkipList.h`

在`memtable.h`中以 `SkipList<const char*, KeyComparator>`的形式被调用。

### 模块接口

```c++
  // Insert key into the list.
  // REQUIRES: nothing that compares equal to key is currently in the list.
  void Insert(const Key& key);//插入key
  bool Contains(const Key& key);//判断是否存在
  

  Comparator const compare_;
  Arena* const arena_;  // Arena used for allocations of nodes

  Node* const head_;

  // Modified only by Insert().  Read racily by readers, but stale
  // values are ok.
  std::atomic<int> max_height_;  // Height of the entire list

  // Read/written only by Insert().
  Random rnd_;

SkipList::Iterator
```

### 模块实现

#### RandomHeight

对于一个新增的节点，有3/4的概率height为0，每增高一层概率降低为原来的1/4，即为height高度的概率为$(1/4)^{height-1}*(3/4)$。

#### 创建新节点

```c++
  x = NewNode(key, height);
  for (int i = 0; i < height; i++) {
    // NoBarrier_SetNext() suffices since we will add a barrier when
    // we publish a pointer to "x" in prev[i].
    x->NoBarrier_SetNext(i, prev[i]->NoBarrier_Next(i));//将prev的next设置为自己的next
    prev[i]->SetNext(i, x);//修改prev的next为自己。
  }
```



### 相关语法



### reference 

https://blog.csdn.net/weixin_36145588/article/details/76393448

[时间复杂度分析](https://blog.csdn.net/yaling521/article/details/78130271)

