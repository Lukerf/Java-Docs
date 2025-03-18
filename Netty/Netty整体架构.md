### 网络通信层

#### BootStrap和ServerBootStrap

BootStrap 负责Netty程序的启动、初始化、服务器连接等过程

BootStrap负责客户端的启动，ServerBootStrap负责服务端的启动。ServerBootStrap会绑定两个EventLoopGroup,这两个一个被称为Boss,一个被称为Worker。Boss被负责接收新的连接，然后将连接分配个一个个worker处理连接。就是Reactor模型中MainReactor和SubReactor的关系

#### Channel

Channel提供了基本的 API 用于网络 I/O 操作，如 register、bind、connect、read、write、flush 等，以 JDK NIO Channel 为基础的，相比较于 JDK NIO，Netty 的 Channel 提供了更高层次的抽象，屏蔽了底层Socket的复杂性

### 事件调度层

<img src="https://learn.lianglianglee.com/%e4%b8%93%e6%a0%8f/Netty%20%e6%a0%b8%e5%bf%83%e5%8e%9f%e7%90%86%e5%89%96%e6%9e%90%e4%b8%8e%20RPC%20%e5%ae%9e%e8%b7%b5-%e5%ae%8c/assets/CgqCHl-NPG6APzDfAAbX5ACAFh8001.png" alt="Drawing 4.png" style="zoom:33%;" />

1. 一个 EventLoopGroup 往往包含一个或者多个 EventLoop。EventLoop 用于处理 Channel 生命周期内的所有 I/O 事件，如 accept、connect、read、write 等 I/O 事件。
2. EventLoop 同一时间会与一个线程绑定，每个 EventLoop 负责处理多个 Channel。
3. 每新建一个 Channel，EventLoopGroup 会选择一个 EventLoop 与其绑定。该 Channel 在生命周期内都可以对 EventLoop 进行多次绑定和解绑。

EventLoopGroup和EventLoop 对应到Reactor模型：

1. **单线程模型**：EventLoopGroup 只包含一个 EventLoop，Boss 和 Worker 使用同一个EventLoopGroup；
2. **多线程模型**：EventLoopGroup 包含多个 EventLoop，Boss 和 Worker 使用同一个EventLoopGroup；
3. **主从多线程模型**：EventLoopGroup 包含多个 EventLoop，Boss 是主 Reactor，Worker 是从 Reactor，它们分别使用不同的 EventLoopGroup，主 Reactor 负责新的网络连接 Channel 创建，然后把 Channel 注册到从 Reactor。

### 服务编排层

![Drawing 8.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/CgqCHl-NPK-ADq0pAABb1k5Zwu8681.png)

ChannelPipeline负责组装各种ChannelHandler，内部通过双向链表将不同的 ChannelHandler 链接在一起。

ChannelPipeline 中包含入站 ChannelInboundHandler 和出站 ChannelOutboundHandler，分别处理数据读取和发送事件

ChannelHandlerContext用于保存 ChannelHandler 上下文



### 总结

![Drawing 9.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Ciqc1F-NPLeAPdjRAADyud16HmQ759.png)

- 服务端启动初始化时有 Boss EventLoopGroup 和 Worker EventLoopGroup 两个组件，其中 Boss 负责监听网络连接事件。当有新的网络连接事件到达时，则将 Channel 注册到 Worker EventLoopGroup。
- Worker EventLoopGroup 会被分配一个 EventLoop 负责处理该 Channel 的读写事件。每个 EventLoop 都是单线程的，通过 Selector 进行事件循环。
- 当客户端发起 I/O 读写事件时，服务端 EventLoop 会进行数据的读取，然后通过 Pipeline 触发各种监听器进行数据的加工处理。
- 客户端数据会被传递到 ChannelPipeline 的第一个 ChannelInboundHandler 中，数据处理完成后，将加工完成的数据传递给下一个 ChannelInboundHandler。
- 当数据写回客户端时，会将处理结果在 ChannelPipeline 的 ChannelOutboundHandler 中传播，最后到达客户端。
