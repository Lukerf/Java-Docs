异步机制

Redis服务需要进行的操作

- 客户端进行增删改查操作
- 写后日志，RDB快照，AOF日志
- 主从节点，主库生成RDB快照，发送RDB，从库接受RDB，清空数据库，加载RDB文件
- 切片集群，实例间哈希槽通信，数据迁移

<img src="https://static001.geekbang.org/resource/image/6c/22/6ce8abb76b3464afe1c4cb3bbe426922.jpg?wh=2685*2250" alt="img" style="zoom:25%;" />

**Redis会遇到的阻塞的操作**

- 集合全量删除和聚合操作
- bigkey删除
- 生成RDB快照-fork子进程的时候会阻塞主进程
- 记录AOF日志-AOF日志同步写磁盘
- 清空数据库
- 从库加载RDB文件

能够允许异步的操作，应该不依赖返回结果，以及不是关键路径上的操作

集合全量删除，bigkey删除

AOF日志同步写磁盘

从库加载RDB文件

