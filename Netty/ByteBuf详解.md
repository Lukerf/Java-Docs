ByteBuf是Netty的数据容器，所有网络通行中字节流的传输都是通过ByteBuf完成的。

### 1. JDK NIO包中的ByteBuffer

![Netty11](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Ciqc1F-3ukmAImo_AAJEEbA2rts301.png)

- mark：为某个读取过的关键位置做标记，方便回退到该位置；
- position：当前读取的位置；
- limit：buffer 中有效的数据长度大小；
- capacity：初始化时的空间容量。

存在问题：

1. 无法动态扩容，不好指定大小
2. ByteBuffer只能通过position操作可读写的位置，使用时需要频繁切换读写状态



### 2. ByteBuf

![Netty11（2）.png](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/CgqCHl-3uraAAhvwAASZGuNRMtA960.png)

第一部分是**废弃字节**，表示已经丢弃的无效字节数据。

第二部分是**可读字节**，表示 ByteBuf 中可以被读取的字节内容，可以通过 writeIndex - readerIndex 计算得出。从 ByteBuf 读取 N 个字节，readerIndex 就会自增 N，readerIndex 不会大于 writeIndex，当 readerIndex == writeIndex 时，表示 ByteBuf 已经不可读。

第三部分是**可写字节**，向 ByteBuf 中写入数据都会存储到可写字节区域。向 ByteBuf 写入 N 字节数据，writeIndex 就会自增 N，当 writeIndex 超过 capacity，表示 ByteBuf 容量不足，需要扩容。

第四部分是**可扩容字节**，表示 ByteBuf 最多还可以扩容多少字节，当 writeIndex 超过 capacity 时，会触发 ByteBuf 扩容，最多扩容到 maxCapacity 为止，超过 maxCapacity 再写入就会出错。

#### ByteBuf使用

```java
public class ByteBufTest {

    public static void main(String[] args) {

        ByteBuf buffer = ByteBufAllocator.DEFAULT.buffer(6, 10);

        printByteBufInfo("ByteBufAllocator.buffer(5, 10)", buffer);

        buffer.writeBytes(new byte[]{1, 2});

        printByteBufInfo("write 2 Bytes", buffer);

        buffer.writeInt(100);

        printByteBufInfo("write Int 100", buffer);

        buffer.writeBytes(new byte[]{3, 4, 5});

        printByteBufInfo("write 3 Bytes", buffer);

        byte[] read = new byte[buffer.readableBytes()];

        buffer.readBytes(read);

        printByteBufInfo("readBytes(" + buffer.readableBytes() + ")", buffer);

        printByteBufInfo("BeforeGetAndSet", buffer);

        System.out.println("getInt(2): " + buffer.getInt(2));

        buffer.setByte(1, 0);

        System.out.println("getByte(1): " + buffer.getByte(1));

        printByteBufInfo("AfterGetAndSet", buffer);

        System.out.println("buffer.refCnt(): " + buffer.refCnt());
        buffer.release(); // 释放资源，refCnt()为0表示可以被回收
        System.out.println("buffer.refCnt(): " + buffer.refCnt());
    }

    private static void printByteBufInfo(String step, ByteBuf buffer) {

        System.out.println("------" + step + "-----");

        System.out.println("readerIndex(): " + buffer.readerIndex());

        System.out.println("writerIndex(): " + buffer.writerIndex());

        System.out.println("isReadable(): " + buffer.isReadable());

        System.out.println("isWritable(): " + buffer.isWritable());

        System.out.println("readableBytes(): " + buffer.readableBytes());

        System.out.println("writableBytes(): " + buffer.writableBytes());

        System.out.println("maxWritableBytes(): " + buffer.maxWritableBytes());

        System.out.println("capacity(): " + buffer.capacity());

        System.out.println("maxCapacity(): " + buffer.maxCapacity());

    }

}
```



#### ByteBuf分类

![image](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/Ciqc1F-3h3WAMF4CAAe4IOav4SA876.png)

**Heap/Direct 就是堆内和堆外内存**

**Pooled/Unpooled 表示池化还是非池化内存**

**Unsafe/非 Unsafe 的区别在于操作方式是否安全**

