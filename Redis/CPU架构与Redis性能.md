### CPU多核架构

​	主流服务器上，一个CPU处理器有10到20多个物理核，物理核之间共享一个L3缓存（一般几MB到几十MB，延迟是L1和L2的近10倍），主流的CPU核中，每个物理核运行两个超线程，也叫逻辑核，逻辑核共享L1和L2缓存（一般只有KB级别）。

<img src="https://static001.geekbang.org/resource/image/d9/09/d9689a38cbe67c3008d8ba99663c2f09.jpg?wh=3065*1633" alt="img" style="zoom:25%;" />



### 多CPU架构

<img src="https://static001.geekbang.org/resource/image/5c/3d/5ceb2ab6f61c064284c8f8811431bc3d.jpg?wh=3000*1252" alt="img" style="zoom:25%;" />