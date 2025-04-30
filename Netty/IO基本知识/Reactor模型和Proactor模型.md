### 1. Reactor模型

#### 2.1 3种Reactor线程模型

1. 单Reactor 单线程/进程

   适用于低并发场景

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZcvaLVakREneqAvT5K3bknhx9Rl8jEhn0sLY5qEFPz7wNkLIkIu4fxEWrZcx2N8hpXOd23ZtfkyibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- Reactor 对象的作用是监听和分发事件；

- Acceptor 对象的作用是获取连接；

- Handler 对象的作用是处理业务；

  

2. 单Reactor 多线程/进程

   默认模式，平衡吞吐与延迟

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZcvaLVakREneqAvT5K3bknhxqr0wQngia4aqVnGbp5czApKH9SJOvRjMglS9fgxzZ2t8QFhq8E634Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3. 多Reactor 多线程/进程

   支持自定义线程池隔离I/O与业务逻辑，承载海量客户端连接建立以及I/O操作

![图片](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZcvaLVakREneqAvT5K3bknhS0hQ9ibrBOtDBQGkNQgTaD0BwpoibrVwv9Uq4hbEP3VGUcYChoia3UjmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



#### 2. 2 Reactor高性能的原因

1. 基于I/O多路复用
2. 主从Reactor多线程模型解决了高并发场景下单个NIO线程无法承载海量客户端连接建立以及I/O操作的性能瓶颈
3. 

### 2. Proactor模型

- **Proactor 是异步网络模式， 感知的是已完成的读写事件**。在发起异步读写请求时，需要传入数据缓冲区的地址（用来存放结果数据）等信息，这样系统内核才可以自动帮我们把数据的读写工作完成，这里的读写工作全程由操作系统来做，并不需要像 Reactor 那样还需要应用进程主动发起 read/write 来读写数据，操作系统完成读写工作后，就会通知应用进程直接处理数据。



总结：无论是 Reactor，还是 Proactor，都是一种基于「事件分发」的网络编程模式，区别在于 Reactor 模式是基于「待完成」的 I/O 事件，而 Proactor 模式则是基于「已完成」的 I/O 事件。