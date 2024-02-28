- noeviction: 不淘汰
- volatile-random: 设置了过期时间的，随机淘汰
- volatile-ttl：设置了过期时间的，淘汰剩余存活时间最短的
- volatile-lru：设置了过期时间的，淘汰最近未访问的数据，redisObject中的lru字段，3字节存储上一次访问的时间戳，会淘汰lru最小的数据。这
- volatile-lfu：设置了过期时间的，最近最少访问。
- allkeys-random：同volatile，只是淘汰范围改为了全部数据，而不是只有设置了过期时间的
- allkeys-lru：
- allkeys-lfu：



### lfu策略详解：

lru也能淘汰最近没有访问的数据，但是如果一个数据在访问一次以后，再也没有访问，那么还是需要过很长一段时间，直到redisObject中lru值在所有数据中是很小的时候才会淘汰，这段时间都是属于缓存被污染了，为了解决这个问题，需要用到lfu。

lfr: 淘汰设置了过期时间的key中，最近最少访问。原理是redisObject中的lru字段包含2字节的ldt值（记录上次访问时间戳）和1字节的counter值（记录数据的访问次数），先判断counter值，counter值相等，再淘汰ldt最小的值。

counter值的增加：并不是访问一次就+1，因为1个字节最多只有到255，计算规则是通过配置项lfu_log_factor控制计数器的增加的速度，这种方式计数并不是完全精确，只是访问次数多的，大概率counter值更大

counter值的衰减：（当前时间-ldt）/ lfu_decay_time值，这个就是counter要衰减的值，所以lfu_decay_time配置项越大，衰减越慢