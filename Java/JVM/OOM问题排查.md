### 1. 获取日志

手动获取： 

自动获取：启动参数配置 --XX: +HeapDumpOnOutOfMemoryError



### 2. 内存溢出问题原因

1. JVM堆内存设置不合理，通过-Xms设置最小值，-Xmx: 设置最大值
2. 代码中出现大量大对象不能被回收

### 3. 常见的几种内存泄漏的情况

1. 



### 4. 实战案例分析



![image-20250430164309925](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250430164309925.png)





![image-20250430170026437](https://raw.githubusercontent.com/Lukerf/Java-Docs/master/image/image-20250430170026437.png)