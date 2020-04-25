# Gemini

## 源码阅读

### 提供的功能(接口)

```c++
  //返回线程所属的socket
  inline int get_socket_id(int thread_id) 

  //返回线程在socket中的id
  inline int get_socket_offset(int thread_id) {

  void init() 

  void fill_vertex_array(T * array, T value)
  
  //使用mmap和numa_tonode_memory申请大小为vertices的数组，并且每个cpu能够接触的内存部分是不同的。
  //使用local_partition界定各个cpu能接触到的不同节点
  T * alloc_vertex_array() 
  
  //释放申请的arrary
  T * dealloc_vertex_array(T * array)

  //
  T * alloc_interleaved_vertex_array()

  //将vertex array 结果输出到path位置的外存
  //每个partition只会写入自己管理的vertex部分(即alloc_vertex_array中的部分)
  //
  //由partitionid = 0的进程创建文件
  void dump_vertex_array(T * array, std::string path) {
      
      
  //将外存path位置数据的的存入到vertex array
  //每个partition只会读入入自己管理的vertex部分
  void restore_vertex_array(T * array, std::string path) {

  //VertexSubset 是一个bitmap，其实是new一个bitmap
  VertexSubset * alloc_vertex_subset() 
      
      
  //查找某个节点所在的partition(多机)
  int get_partition_id(VertexId v_i)
      
  //查找某个节点所在的numa 的local partition
  int get_local_partition_id(VertexId v_i)
      
      
  
  void load_undirected_from_directed(std::string path, VertexId vertices) {
  void load_directed(std::string path, VertexId vertices) {

      
  //交换每条边的方向
  void transpose() 
      
      
  //load graph的时候使用
  void tune_chunks()

  // active（bitmap）代表选择的节点1代表选择0代表未选。只对每个为1的节点执行。
  // 最终reduce返回结果R
  //process_vertices 代码较短，内容比较直接分为1.线程状态初始化，2.
  R process_vertices(std::function<R(VertexId)> process, Bitmap * active) {
     
        
  void flush_local_send_buffer(int t_i) {
  void emit(VertexId vtx, M msg) {
      
     
    R process_edges(std::function<void(VertexId)> sparse_signal, std::function<R(VertexId, M, VertexAdjList<EdgeData>)> sparse_slot, std::function<void(VertexId, VertexAdjList<EdgeData>)> dense_signal, std::function<R(VertexId, M)> dense_slot, Bitmap * active, Bitmap * dense_selective = nullptr) {


```



```c++
  //MPI多机相关
  int partition_id;
  int partitions;

  size_t alpha;

  int threads;//线程个数
  int sockets;//numa 的sockets 个数
  int threads_per_socket;

  size_t edge_data_size;
  size_t unit_size;
  size_t edge_unit_size;

  bool symmetric;
  
  //全局信息
  VertexId vertices;//vertexId(uint32_t) 全局的节点数目，与局部的owned_vertices相对
  EdgeId edges;//EdgeId(uint64_t)
  VertexId * out_degree; // VertexId [vertices]; numa-aware
  VertexId * in_degree; // VertexId [vertices]; numa-aware


//存储节点区间的数组partition_offset管理多机的信息，local_partition_offset管理numa的多cpu信息
  VertexId * partition_offset; // VertexId [partitions+1]
  VertexId * local_partition_offset; // VertexId [sockets+1]



 //局部信息
  VertexId owned_vertices;
  EdgeId * outgoing_edges; // EdgeId [sockets]
  EdgeId * incoming_edges; // EdgeId [sockets]


//
  Bitmap ** incoming_adj_bitmap;
  EdgeId ** incoming_adj_index; // EdgeId [sockets] [vertices+1]; numa-aware
  AdjUnit<EdgeData> ** incoming_adj_list; // AdjUnit<EdgeData> [sockets] [vertices+1]; numa-aware
  Bitmap ** outgoing_adj_bitmap;
  EdgeId ** outgoing_adj_index; // EdgeId [sockets] [vertices+1]; numa-aware
  AdjUnit<EdgeData> ** outgoing_adj_list; // AdjUnit<EdgeData> [sockets] [vertices+1]; numa-aware

  VertexId * compressed_incoming_adj_vertices;
  CompressedAdjIndexUnit ** compressed_incoming_adj_index; // CompressedAdjIndexUnit [sockets] [...+1]; numa-aware
  VertexId * compressed_outgoing_adj_vertices;
  CompressedAdjIndexUnit ** compressed_outgoing_adj_index; // CompressedAdjIndexUnit [sockets] [...+1]; numa-aware


//线程信息
  ThreadState ** thread_state; // ThreadState* [threads]; numa-aware
  ThreadState ** tuned_chunks_dense; // ThreadState [partitions][threads];
  ThreadState ** tuned_chunks_sparse; // ThreadState [partitions][threads];

  size_t local_send_buffer_limit;
  MessageBuffer ** local_send_buffer; // MessageBuffer* [threads]; numa-aware

  int current_send_part_id;//partitions
  MessageBuffer *** send_buffer; // MessageBuffer* [partitions] [sockets]; numa-aware
  MessageBuffer *** recv_buffer; // MessageBuffer* [partitions] [sockets]; numa-aware
```

### 具体实现

#### bitmap

```c++
  size_t size;
  unsigned long * data; //注意在linux下unsigned long是64位windows下是32位，这个代码只支持linux下的运行，感觉替换成uint32_t比较合适
```

#### process_edge

#### 数据加载



## 论文资料

