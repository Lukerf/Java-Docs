### 1. 获取日志

手动获取： 

自动获取：启动参数配置 --XX: +HeapDumpOnOutOfMemoryError



### 2. 内存溢出问题原因

1. JVM堆内存设置不合理，通过-Xms设置最小值，-Xmx: 设置最大值
2. 代码中出现大量大对象不能被回收

### 3. 常见的几种内存泄漏的情况

1. 



### 4. MAT工具分析按钮操作



![image-20250506160446082](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250506160446082.png)



![image-20250430164309925](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250430164309925.png)





![image-20250430170026437](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250430170026437.png)

##### 1. List objects(列出对象引用)

- **with outgoing references**：当前对象引用了哪些对象（" outgoing "指向外部的引用）。
- **with incoming references**：哪些对象引用了当前对象（" incoming "指向自己的引用）。

##### 2. Path to GC Roots(到 GC Roots的引用路径)

显示从当前对象到GC Root(垃圾回收根节点)的完整引用链

##### 5. Show Retained Sed (显示保留集)

显示当前对象直接或间接支配的所有对象（即如果该对象被回收，这些对象也会被回收）

**三者的核心区别**

| 功能                  | 分析方向                 | 典型应用场景                       |
| :-------------------- | :----------------------- | :--------------------------------- |
| **Path to GC Roots**  | 从对象向上追溯到 GC Root | 为什么对象泄漏？谁在阻止它被回收？ |
| **List objects**      | 直接引用关系（出/入）    | 对象内部结构或直接引用者是谁？     |
| **Show Retained Set** | 对象支配的所有下级对象   | 对象实际占用了多少内存？           |



### 5. 实战案例分析

打开hprof文件，点击 “ Leak Suspects ” 查看内存泄漏分析，可以看到pdfToImg线程池创建的线程占用了78.8的内存

![image-20250506160643848](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250506160643848.png)

任意找一个pdfToImg的Thread，查看其引用的对象（out），哪个大对象没有释放

![image-20250506160923137](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250506160923137.png)

发现大对象是BufferedImage

![image-20250506175431486](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250506175431486.png)

右键当前Thread->Java Basics -> Thread Details，可以看到当前线程的调用栈，代码里面发现有PdfToImgTask方法，查看代码，发现是这个方法里创建的BufferedImage没有调用flush进行回收

![image-20250506175306902](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250506175306902.png)



![image-20250506175837951](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250506175837951.png)

以Thread持有BufferedImage为例：

1. 在MAT中找到问题Thread对象
2. 查看stackTrace定位最近执行的业务方法
3. 检查threadLocals中是否缓存了图像
4. 分析target Runnable对象的成员变量
5. 结合代码搜索确认具体创建位置
6. 检查是否缺少flush()或clear()调用