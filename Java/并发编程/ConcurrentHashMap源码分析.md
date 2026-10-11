### 为什么不用HashTable

HashTable效率低下，因为其实现使用了synchronized对put操作进行加锁，加锁时对整个对象加锁，并发度只有1。

### 1.7版本的ConcurrentHashMap

基于分段锁实现的，结构是一个Segment数组，而每个Segment都相当于一个HashMap，Segment通过继承ReentrantLock进行加锁，也就是说并发量就是Segment数组的大小，且Segment数组不可扩容。

#### 核心思想

通过key的hash值定位到segment，segment里面有一个HashEntry数组，通过Hash计算HashEntry，HashEntry是链表的表头，再顺着链表找插入的位置（无红黑树结构）

#### 关键代码

构造函数

ConcurrentHashMap(int initialCapacity,float loadFactor,int concurrencyLevel)。入参有

初始容量：整个ConcurrentHashMap的初始容量，实际操作的时候需要平均分给每个Segment

负载因子: Segment内部HashMap的负载因子

concurrencyLevel: 并发量，实际就是segment的大小



### 1.8版本的ConcurrentHashMap

结构上和HashMap一致，通过数组+链表+红黑树实现

数据安全通过对Node节点加锁实现，使用Synchronized加锁，如果Node节点为空，则通过CAS操作赋值

ConcurrentHashMap用的头插法，插入效率为O(1)



**数据结构和重要变量**



**get方法**

get方法没有加锁，提升了性能，线程安全是因为对Node节点的val和next属性都加了volatile修饰，保证了可见性



**put方法**

1. 计算 key 的 hash。
2. `for` 循环自旋，直到操作成功。
3. 检查 table 是否为空，为空则用 **CAS** 初始化。
4. 定位的桶为空，则用 **CAS** 尝试插入新节点。
5. 如果桶的头节点 hash 为 `MOVED`（-1），说明正在扩容，当前线程会**协助扩容**（这是重点，体现了并发协作）。
6. 如果桶不为空，用 **`synchronized`** 锁住头节点，然后执行链表插入或红黑树插入。
7. 最后调用 `addCount` 更新计数，并检查是否需要扩容



**扩容**

触发扩容的条件

1. 检测到元素数量超过sizeCtl(数组大小*负载因子)
2. 链表过长，但是数组太小，某个链表的长度超过8，但是数组大小小于64时



#### 问题

1. 为什么JDK 1.7 的 `ConcurrentHashMap` 选择分段锁，而非一开始就用 `synchronized` 锁节点

核心原因在于 **JDK 1.6 之前，`synchronized` 还是一个不折不扣的“重量级锁”**，直接锁节点的开销在当时是无法接受的。

JDK1.6版本引入了锁升级（偏向锁->轻量级锁->重量级锁）以及锁消除、锁粗化等编译期优化
