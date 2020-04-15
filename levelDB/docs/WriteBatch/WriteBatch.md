# WriteBatch

### 模块概要

`include/leveldb/write_batch.h`
`db/write_batch_internal.h`
`db/write_batch.cc`

进行批量写入的结构。


### 模块接口

```c++
//writeBatch
Put()//存储 put 操作
Delete()//存储 delete 操作
Iterate()//遍历内容 经handler接口 存到 memtable 中
   
//writeBatchInternal
WriteBatchInternal::Count()：//返回个数
WriteBatchInternal::SetCount()//将rep_的后四位encode。
WriteBatchInternal::Sequence()：//返回 sequence number
WriteBatchInternal::SetSequence()//将rep_的前8位encode。
WriteBatchInternal::Contents()：//返回内容
WriteBatchInternal::SetContents() //替换writeBatch中的rep_新的contents
WriteBatchInternal::Append(WriteBatch* dst, const WriteBatch* src)：//拼接两个 WriteBatch
WriteBatchInternal::InsertInto(const WriteBatch* b, MemTable* memtable)：//将 WriteBatch 写入 memtable，转发给了 WriteBatch::Iterate()

    
//MemTableInserter
  SequenceNumber sequence_;
  MemTable* mem_;//TODO(MemTable)
Put()//mem_新增一个put的log
Delete()//mem_新增一个delete的log
```

### 模块实现

rep_首部存放着`kHeader`的(`kHeader` = 12)大小的数据。8字节的squenceNumber和4字节的count。后面存放的是batch中的数据。包括(`kTypeDeletion`|`kTypeValue`) 和`key `，`value`。

```c++
//存储格式 
//WriteBatch::rep_ :=
//      sequence: fixed64
//      count: fixed32
//      data: record[count]
// record :=
//      kTypeValue varstring varstring 
//      kTypeDeletion varstring
// varstring :=
//      len: varint32
//      data: uint8[len]
```



#### iterate

`WriteBatchInternal::InsertInto`调用`iterate`，iterate调用`coding`中的方法（`GetLengthPrefixedSlice`）来解析rep\_中的，然后写入到`MemTableInserter`中的`mem_`。



 

> **(转)**为什么 leveldb 采用了这样的其中一个线程去批量操作而其他线程进行等待的方式呢？我们知道 leveldb 在实际插入过程中会有一系列的判断，和写日志到磁盘的操作。而首先这些判断都是在保持锁的情形下进行的，这里其实也就注定了只能是串行的；其次对于写磁盘，将多次写合并为一次写入会显著的提高效率，相当于 N * 磁盘寻址时间 + 写入时间总和变为了 一次磁盘寻址时间 + 写入时间总和，即节省了（N - 1）次磁盘寻址时间。

 

### 相关依赖

上层方法调用：当插入数据时，`DBImpl::Put()` 和 `DBImpl::Delete()` 调用 `WriteBatch::Put()` 和 `WriteBatch::Delete()` 方法将记录添加进 `WriteBatch::rep_`，然后 `DBImpl::Write(WriteBatch*)` 调用 `WriteBatchInternal::InsertInto()` 方法。

### reference

[https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/DB/WriteBatch%20-%202018-10-01%20-%20rsy.md](https://github.com/rsy56640/read_and_analyse_levelDB/blob/master/architecture/DB/WriteBatch - 2018-10-01 - rsy.md)