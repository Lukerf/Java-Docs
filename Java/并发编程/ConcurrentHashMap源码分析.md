### 为什么不用HashTable

HashTable效率低下，因为其实现使用了synchronized对put操作进行加锁，加锁时对整个对象加锁，并发度只有1。

### 1.7版本的ConcurrentHashMap

结构是一个Segment数组，而每个Segment都相当于一个HashMap，Segment通过继承ReentrantLock进行加锁，也就是说并发量就是Segment数组的大小，且Segment数组不可扩容。

#### 核心思想

通过key的hash值定位到segment，segment里面有一个HashEntry数组，通过Hash计算HashEntry，HashEntry是链表的表头，再顺着链表找插入的位置（无红黑树结构）

#### 关键代码

构造函数

ConcurrentHashMap(int initialCapacity,float loadFactor,int concurrencyLevel)。入参有

初始容量：整个ConcurrentHashMap的初始容量，实际操作的时候需要平均分给每个Segment

负载因子: Segment内部HashMap的负载因子

concurrencyLevel: 并发量，实际就是segment的大小

