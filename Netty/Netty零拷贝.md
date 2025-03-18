### 1. 零拷贝概念

在进行数据操作时，不需要将数据从一个内存位置拷贝到另一个内存位置，这样可以减少一次内存拷贝的损耗。节省CPU时钟周期和内存带宽



### 2. 传统零拷贝

![Drawing 0.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Ciqc1F_Qbz2AD4uMAARnlgeSFc4993.png)

从磁盘读取数据，发送到网络上，需要进行四次拷贝：磁盘-->内核缓冲区-->用户缓冲区-->socket缓冲区--拷贝到协议引擎

内核缓冲区：可以充当缓存的作用，主要是可以实现文件的预读（一次性读取比请求更多的数据，请求可以先从缓存获取数据，获取不到再从磁盘获取），以及延迟回写（不直接写入磁盘，数据堆积后再写入）

传统零拷贝技术：Linux中系统调用sendfile()可以将数据从一个文件描述符传输到另一个文件描述符。Java中的零拷贝技术就是依赖了操作系统零拷贝的机制。

![Drawing 1.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/CgqCHl_Qb0mANyjrAATEtVu9f6c390.png)

Linxu2.4版本之后，通过追加一些Descriptor信息，消除了最后一次CPU拷贝

![Drawing 2.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/CgqCHl_Qb2eASFBJAAT4WPf__Us976.png)

零拷贝并不是完全不存在数据拷贝，剩下的两次都是硬件完成，不需要CPU参与。



### 3. Netty的零拷贝技术



#### CompositeByteBuf

普通ByteBuf对象合并

```
ByteBuf httpBuf = Unpooled.buffer(header.readableBytes() + body.readableBytes());
httpBuf.writeBytes(header);
httpBuf.writeBytes(body);
```

CompositeByteBuf由多个ByteBuf组成，但是逻辑上仍然组成一个整体，里面包括一个Component数组

```
CompositeByteBuf httpBuf = Unpooled.compositeBuffer();
httpBuf.addComponents(true, header, body);
```

![Drawing 3.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Ciqc1F_Qb3SAP4vUAAZG1WvALhY410.png)

![Drawing 4.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Ciqc1F_Qb3yAUwbLAAVl7ZwmfJ0669.png)

