# 1. key-value存储结构

所有的k-v都由一个全局哈希表维护，数据结构是一个哈希桶，哈希桶的冲突解决方案是链表法

每个entry包括3个8B的指针,key,value和下一个entry，由于redis的内存分配库jemalloc分配内存时，会找最近的2N，也就是32，所以entry大小为32B

key和value的指针都指向RedisObject，RedisObject包括8B的元数据，元数据中会存储type(数据类型，String,List等),encoding（对象的编码方式）,lru（最近一次被访问时间 3B）,refcount（对象的引用计数）

int 编码模式：Long型整数，则直接存储值。

embstr编码模式：字符串小于44字节，SDS和RedisObject是一块连续的空间，这样避免内存碎片。

raw编码模式：字符串大于44字节，SDS和RedisObject不是连续空间

![未命名文件](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/未命名文件.jpg)

案例：一个8字节的key，和一个8字节的long型整数，需要占用多少空间呢

结果：需要64字节，如果需要存储大量的String时，这种数据类型会很浪费内存空间

# 2. 各数据结构和类型使用场景

## 2.1  存储大量key 



### 2.1.1 压缩列表

#### 使用场景

如果保存的键值对本身占用空间不大时，但是数量很多时，如果用String存储，元数据会占用很多的空间，这个时候数据结构可以选择用压缩列表存储，比如Hash,List,Sorted Set都有用到压缩列表，这几个数据类型可以选择使用

#### 实现

表头有三个字段 zlbytes、zltail 和 zllen，分别表示列表长度、列表尾的偏移量，以及列表中的 entry 个数。压缩列表尾还有一个 zlend，表示列表结束。每个entry的内容如下

prev_len: 表示前一个entry的长度，如果前一个entry长度小于254（zlend是255，用来表示结束），那么用一个字节存储，如果前一个字段的长度大于254，用5个字节存储

encoding: 编码类型，1个字节

len：表示自身长度，4个字节

key：存储的内容

<img src="https://static001.geekbang.org/resource/image/f6/9f/f6d4df5f7d6e80de29e2c6446b02429f.jpg?wh=3457*972" alt="img" style="zoom:25%;" />

优点：每个entry间紧凑排列，不需要额外的prev、next指针来维护entry链表，大大节省了内存



### 2.1.2Hash类型

#### 实现

默认使用压缩列表实现，当field-value对数量大于hash-max-ziplist-entries或者单个元素的长度大于hash-max-ziplist-value时，会改用哈希表的实现，哈希表的实现就没有那么高效了

- hash-max-ziplist-entries：表示用压缩列表保存时哈希集合中的最大元素个数。
- hash-max-ziplist-value：表示用压缩列表保存时哈希集合中单个元素的最大长度。



Hash类型使用了压缩列表，通过key定位到对应的压缩列表以后，在压缩列表中遍历field，直到找到对应的field后返回，所以hash-max-ziplist-entries不宜设置过大，否则性能下降会很厉害

## 2.2 数据统计

<img src="https://static001.geekbang.org/resource/image/c0/6e/c0bb35d0d91a62ef4ca1bd939a9b136e.jpg?wh=2866*1739" alt="img" style="zoom:25%;" />

#### 聚合计算

场景

1. 统计每天的新增用户数
2. 统计每天的用户留存数（前一天登录了，当天接着登录了的）

```
# 计算并集，并放入到user:id这个set中
SUNIONSTORE  user:id  user:id  user:id:20200803 
# 计算差集，如果user:id:20200804这个Set中的元素，不在user:id这个Set中，则把这个元素记录到user:new这个Set中
SDIFFSTORE  user:new  user:id:20200804 user:id  
# 计算交集，在user:id:20200803这个Set和user:id:20200804中都出现的元素，放到user:id:rem这个Set中
SINTERSTORE user:id:rem user:id:20200803 user:id:20200804
```

**注意事项**

1. 进行聚合计算时，计算复杂度较高，在数据量较大的情况下，可能会阻塞主线程，一般是建议在从库执行聚合计算，或者读取完数据以后在客户端进行计算
2. 切片集群下，如果要统计的多个key在不同的实例中，那么是不能进行计算的，会直接报错。

#### 排序统计

场景问题： 按评论的发布时间进行排序，分页展示评论

List: 按插入顺序排序

Sorted Set: 按设置的权重进行排序

List和Sorted Set的区别: List 是通过元素在 List 中的位置来排序的，当有一个新元素插入时，原先的元素在 List 中的位置都后移了一位。Sorted Set 它是根据元素的实际权重来排序和获取数据的。

#### 二值统计

场景问题：10天中，统计1亿个用户在这10天里连续签到的用户数量

**Bitmap**

是用 String 类型作为底层数据结构实现的一种统计二值状态的数据类型。String 类型是会保存为二进制的字节数组，所以，Redis 就把字节数组的每个 bit 位利用起来，用来表示一个元素的二值状态。

```
# 设置uid:sign:3000:202008 这个bitmap 偏移量为2的bit为1
SETBIT uid:sign:3000:202008 2 1  
#获取
GETBIT uid:sign:3000:202008 2 
#计数 uid:sign:3000:202008这个map中为1 的数量
BITCOUNT uid:sign:3000:202008
# 还支持按位与操作等
```

#### 基数统计

基数统计就是指统计一个集合中不重复的元素个数

也可以用Set实现，但是当数据量很大的时候，会占用大量的空间

HyperLogLog：一种用于统计基数的数据集合类型，它的最大优势就在于，当集合元素数量非常多时，它计算基数所需的空间总是固定的，而且还很小
