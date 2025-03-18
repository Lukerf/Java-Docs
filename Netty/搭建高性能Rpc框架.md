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



### RPC调用方式

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



### 线程模型

![Lark20210101-221822.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Cip5yF_vL7eANr2BAAI2Xxyw-pg628.png)

### 动态代理



### 4. 负载均衡

