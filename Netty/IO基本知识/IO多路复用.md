阻塞IO处理客户端请求

```java
listenfd = socket();   // 打开一个网络通信端口
bind(listenfd);        // 绑定
listen(listenfd);      // 监听
while(1) {
  connfd = accept(listenfd);  // 阻塞建立连接
  int n = read(connfd, buf);  // 阻塞读数据
  doSomeThing(buf);  // 利用读到的数据做些什么
  close(connfd);     // 关闭连接，循环等待下一个连接
}
```



read()方式是阻塞式IO，

为了能够不阻塞当前线程执行

1. 另外开辟线程执行read操作，和doSomething，这样就不阻塞其他连接进行。
2. 但是方式1 中read函数还是阻塞的，如果连接很多时，会达到资源瓶颈
3. 操作系统层面提供非阻塞的读取函数select，select可以读取多个文件描述符connfd，如果这个list中有数据已经准备好，即已经将数据拷贝到了内核空间，操作系统会给准备就绪的文件描述符做上标识，用户层读取到标识不为-1时，则可以接着调用read函数读取数据

```java
while(1) {
  nready = select(list);
  // 用户层依然要遍历，只不过少了很多无效的系统调用
  for(fd <-- fdlist) {
    if(fd != -1) {
      // 只读已就绪的文件描述符
      read(fd, buf);
      // 总共只有 nready 个已就绪描述符，不用过多遍历
      if(--nready == 0) break;
    }
  }
}
```

4. poll和epoll

   poll去除了select只能监听1024个文件描述符的限制

   epoll

    1. 不需要传入list,只需传入每次修改的文件描述符即可

    2. 不再轮询fdlist，而是通过异步IO事件唤醒，内核会将有IO事件的文件描述符返回

       