### 使用

对普通方法加对象锁

对静态方法加类锁

代码块可以指定类锁或者对象锁

### 加锁和释放锁原理

基于对象实现，对象在堆内存中包含了**对象头**，而对象头有**MarkWord**

<img src="E:\学习资料\笔记\Java-Docs\image\对象在堆内存.png" style="zoom:50%;" />

MarkWord中记录了锁的类型和状态，状态有：无锁，偏向锁，轻量级锁，重量级锁（jdk6之前只有有锁和无锁两种），申请锁，锁升级时都会读取MarkWord中的信息

<img src="E:\学习资料\笔记\Java-Docs\image\MarkWord.png" style="zoom:50%;" />

**重量级锁**直接指向C++中的实现：即竞争锁的线程个数，**当前锁的owner，等待被唤醒的线程列表**等信息。

### 

monitorenter是锁计数器+1，monitorexit使锁计数器-1，重入就是一直monitorenter，获取锁失败的线程，也会放入到一个同步队列当中，monitorexit后会通知同步队列出列。







### Synchronized和Lock比较

1. synchronized锁释放只有在代码执行完成或者异常退出以后才会释放锁。Lock可以中断和设置超时
3. synchronized使用不够灵活，只能对类或者对象加锁。而Lock可以自主决定什么时候加锁，什么时候释放锁
4. synchronized不知道是否成功获得锁，Lock可以通过tryLock尝试获取锁，返回一个boolean值