所有的k-v都由一个全局哈希表维护，数据结构是一个哈希桶，哈希桶的冲突解决方案是链表法

每个entry包括3个8B的指针,key,value和下一个entry，由于redis的内存分配库jemalloc分配内存时，会找最近的2N，也就是32，所以entry大小为32B

key和value的指针都指向RedisObject，RedisObject包括8B的元数据，元数据中会存储type(数据类型，String,List等),encoding（对象的编码方式）,lru（最近一次被访问时间 3B）,refcount（对象的引用计数）

int 编码模式：Long型整数，则直接存储值。

embstr编码模式：字符串小于44字节，SDS和RedisObject是一块连续的空间，这样避免内存碎片。

raw编码模式：字符串大于44字节，SDS和RedisObject不是连续空间

![未命名文件](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/未命名文件.jpg)

案例：一个8字节的key，和一个8字节的long型整数，需要占用多少空间呢

结果：需要64字节，如果需要存储大量的String时，这种数据类型会很浪费内存空间

