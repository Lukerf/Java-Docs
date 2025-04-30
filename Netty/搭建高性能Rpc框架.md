### 1. 架构设计

服务提供者

服务消费者

注册中心



### 2. 服务注册与发现

服务提供者：发布服务到注册中心

服务消费者：订阅注册中心，调用时



### 3. 通行协议与序列化

构建通信协议：包括 魔数、协议版本号、序列化算法、报文类型、状态码、requestId、请求数据长度、请求数据

序列化：将请求数据进行序列化处理

编码：将请求体编码成ByteBuf

解码：将ByteBuf解码成请求体，报文类型有request和response，需要解析成不同的数据类型

![Drawing 2.png](https://learn.lianglianglee.com/%e4%b8%93%e6%a0%8f/Netty%20%e6%a0%b8%e5%bf%83%e5%8e%9f%e7%90%86%e5%89%96%e6%9e%90%e4%b8%8e%20RPC%20%e5%ae%9e%e8%b7%b5-%e5%ae%8c/assets/CgqCHl_1MbmAeZgjAAd9EAWpmuE609.png)



### 4. RPC调用方式

Sync同步调用：客户端发起调用以后，当前线程一直阻塞，等待回传或超时异常

Future异步调用：客户端发起调用以后，会拿到一个Future对象，可以通过查询Future对象操作是否完成

```java
// 发起异步调用
Future<Response> future = rpcClient.invokeAsync(request);
// 阻塞等待结果
Response response = future.get();
// 或者轮询检查是否完成
while (!future.isDone()) {
    // 做一些其他事情
}
Response response = future.get();
```

Callback回调调用：客户端发起调用以后无需等待。服务端响应或者超时以后回调callback方法

```java
// 发起异步调用，传入回调函数
rpcClient.invokeAsync(request, new Callback<Response>() {
    @Override
    public void onSuccess(Response response) {
        // 处理成功结果
    }
    @Override
    public void onFailure(Throwable throwable) {
        // 处理失败情况
    }
});
```

Onewany单向调用：客户端发起请求后直接返回，忽略返回结果



### 5. 线程模型

![Lark20210101-221822.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Cip5yF_vL7eANr2BAAI2Xxyw-pg628.png)

### 6. 动态代理



### 7. 负载均衡

ServiceInstance 对象代表一个服务实例，它包含名称 name、唯一标识 id、地址 address、端口 port 以及用户自定义的可选属性 payload。 在Zookeeper服务器中的存储形式如下： 所以可以通过serviceName，获取所有提供服务的服务器信息list 。 通过负载均衡策略（比如一致性哈希环，通过TreeMap存储），选择一个服务实例返回。

```java
Collection<ServiceInstance<ServiceMeta>> serviceInstances = serviceDiscovery.queryForInstances(serviceName);
```

<img src="https://learn.lianglianglee.com/专栏/Netty 核心原理剖析与 RPC 实践-完/assets/CgpVE1_-X2qAZ0QwAALfB-0Ouy4852.png" alt="Drawing 0.png" style="zoom:33%;" />





### 8. 框架优化

#### 8.1 性能优化

##### 8.1.1 注册中心

注册中心的CAP抉择

服务下线以后，为了通知消费者更换其他服务器，采用主动通知+心跳检测的方案。

##### 8.1.2 I/O模型

Netty提供了多Reactor多线程模型

##### 8.1.3 Netty网络参数配置

 ChannelOption 的相关参数，例如：

TCP_NODELAY：是否开启 Nagle 算法。Nagle 算法通过缓存的方式将网络数据包累积到一定量才会发送，

SO_KEEPALIVE：连接保活。启用了 TCP SO_KEEPALIVE 属性，TCP 会主动探测连接状态

##### 8.1.4 序列化方式

选择适当的序列化算法

##### 8.1.5 内存管理



#### 8.2 高可用



