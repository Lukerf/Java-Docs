# 1. Redis Cluster

## 1.1  整体架构

​							<img src="https://static001.geekbang.org/resource/image/79/26/793251ca784yyf6ac37fe46389094b26.jpg?wh=2501*1342" alt="img" style="zoom: 25%;" />

## 1.2 数据在集群的分布

Hash-based: 根据键值对的key，计算一个值以后对哈希槽数目取模（默认为16384个Slot)，得到具体的Slot，好处是使用这种方式利于Redis的均衡分布，不太容易出现数据倾斜。

### 1.2.1 哈希槽的分配

- 使用 cluster create 命令创建集群，Redis自动平均将哈希槽分配在每个实例上
- cluster meet 命令手动建立实例间的连接，Gossip协议通信，形成集群，再使用 cluster addslots 命令，指定每个实例上的哈希槽个数。

### 1.2.2 客户端怎么知道Slot在哪个实例

1. 每个实例只知道自己分配了哪些哈希槽，然后通过与其他实例建立连接，来完成哈希槽分配信息的扩散，这样每个实例就有所有哈希槽的映射关系了，客户端可以从访问的实例中获取所有实例的哈希槽信息，并缓存在本地

2. 重定向机制，当由于**新增实例，或者哈希槽分配发生变化**时，客户端访问实例时并没有相应的Slot，需要进行访问的重定向

   1. 数据已经迁移完成：这个实例会给客户端返回MOVED命令响应结果，结果中包含了新实例的访问地址，客户端就可以访问新实例，并且更新本地缓存。
   2. 数据还在迁移过程中：实例会给客户端返回ASK命令，告知客户端数据还在迁移中，以及迁移的目标实例。如果客户端直接从目标实例GET数据，由于槽位还没有迁移，目标实例会返回-MOVED错误。所以客户端会先给目标实例发送一个ASKING命令，告知目标实例下一条指令不能不理，当做自己的槽位来处理

   ```
   # 数据已经迁移完成
   GET hello:key
   (error) MOVED 13320 172.16.19.5:6379
   
   # 数据还在迁移中
   GET hello:key 
   (error) ASK 13320 172.16.19.5:6379
   ```

# 2. Codis

## 2.1 Codis 的整体架构

- codis server：这是进行了二次开发的 Redis 实例，其中增加了额外的数据结构，支持数据迁移操作，主要负责处理具体的数据读写请求。
- codis proxy：接收客户端请求，并把请求转发给 codis server。
- Zookeeper 集群：保存集群元数据，例如数据位置信息和 codis proxy 信息。
- codis dashboard 和 codis fe：共同组成了集群管理工具。其中，codis dashboard 负责执行集群管理工作，包括增删 codis server、codis proxy 和进行数据迁移。而 codis fe 负责提供 dashboard 的 Web 操作界面，便于我们直接在 Web 界面上进行集群管理。

<img src="https://static001.geekbang.org/resource/image/c7/a5/c726e3c5477558fa1dba13c6ae8a77a5.jpg?wh=2913*1835" alt="img" style="zoom: 25%;" />

## 2. 2 数据在集群中的分布

1. 通过codis dashboard给codis server 分配Slot槽，分配后的结果发给codis proxy，codis proxy本地存储一个路由表，也发给Zookeeper集群一份存储
2. 客户端请求codis proxy，将key进行计算以后进行hash取模，得到Slot槽，proxy在根据路由表向对应的server发送请求

<img src="https://static001.geekbang.org/resource/image/d1/b1/d1a53f8b23d410f320ef145fd47c97b1.jpg?wh=2861*1864" alt="img" style="zoom:25%;" />

## 2.3 集群扩展和数据迁移

#### 2.3.1 扩展codis server

1. **集群扩展**
   1. 增加 codis server 服务，通过codis dashboard配置新的server，并且重新分配Slot槽，然后发送到codis proxy 和 Zookeeper

2. **数据迁移**： 按照Slot的粒度进行迁移，源server往目的server发送Slot，目的server收到以后，给源server发送Ack确认，源server删除对应的Slot
   - 同步迁移：发送和等待接受Ack的过程是阻塞的。
   - 异步迁移：
     - 源server把数据发送给目标server以后，就可以处理其他请求了，而且这个过程中，迁移数据会被设置为只读，所以不会出现源server和目标server数据不一致的问题。
     - 对于bigkey，为了避免疑问序列化大量数据造成阻塞源server，源server会分批迁移bigkey。同时，为防止出现一个bigkey中的数据，一部分在源server，一部分在目标server，会将目标server中的数据设置临时过期时间，如果出现故障，导致数据过期，会将目标server中的数据删除，这样就保证了迁移的原子性。

#### 2.3.2 扩展codis proxy 

新增启动codis proxy服务，通过codis dashboard把proxy加入集群，发送zookeeper，并给proxy发送路由表。由于zookeeper中存储了proxy信息，所以客户端可以读取zookeeper，给proxy发请求，

## 2.4集群的可靠性

codis server : 给每个codis server 配置从库，并且也可以利用哨兵机制，监控codis server，和主从集群机制一样，只是每个codis server 都是一个server group

codis proxy 和zookeeper： zookeeper只要保证有半数能工作就能保证工作

codis dashboard： 无状态服务，复杂压力不大，只用于给管理员配置使用，故障以后重启就行。



# 3. Redis Cluster 和 Codis的比较和选择

