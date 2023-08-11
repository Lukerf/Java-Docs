全称：AbstractQueuedSynchronizer

#### 核心思想：

如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用**CLH队列**锁实现的，即将暂时获取不到锁的线程加入到队列中。

#### 两种对资源的共享模式：

Exclusive(独占)：只能一个线程获取资源

- ReentrantLock
- ReentrantReadWriteLock

Share(共享)：可以多个线程执行

- CountDownLatch
- Semaphore
- CyclicBarrier

#### 关键代码

AQS使用一个volatile修饰的int成员变量来表示同步状态

以ReentrantLock为例，**state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1**（通过CAS set的）。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0(即释放锁)为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的(**state会累加**)，通过**exclusiveOwnerThread 是否等于当前线程**，这就是**可重入的概念**。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

```java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

 底层使用了模板方法模式,自定义同步器时需要重写下面几个AQS提供的模板方法

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

核心方法 acquire : 核心就是state能够获取到，为0可以，否则不行。如果是可重入，且线程的owner是当前线程，则也可以获取成功,state+1。

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

![](E:\学习资料\笔记\Java-Docs\image\aqs流程图.png)

首先调用tryAcquire方法，调用此方法的线程会试图在独占模式下获取对象状态。此方法应该查询是否允许它在独占模式下获取对象状态，如果允许，则获取它。在AbstractQueuedSynchronizer源码中默认会抛出一个异常，即需要子类去重写此方法完成自己的逻辑。之后会进行分析。

**若tryAcquire失败，则调用addWaiter方法，addWaiter方法完成的功能是将调用此方法的线程封装成为一个结点并放入Sync queue**

**调用acquireQueued方法，此方法完成的功能是无限循环，直到前一个节点为头节点，且获取共享资源成功，退出无限循环，返回true，否则，返回false。**

```java
final boolean acquireQueued(final Node node, int arg) {
    // 标志
    boolean failed = true;
    try {
        // 中断标志
        boolean interrupted = false;
        for (;;) { // 无限循环
            // 获取node节点的前驱结点
            final Node p = node.predecessor(); 
            if (p == head && tryAcquire(arg)) { // 前驱为头节点并且成功获得锁
                setHead(node); // 设置头节点
                p.next = null; // help GC
                failed = false; // 设置标志
                return interrupted; 
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```



#### ReentrantLock

ReentrantLock的 lock()和unlock()等操作最终都转换为内部类Sync的操作，Sync则又是继承的AQS。上面例子中说了ReentrantLock的工作原理

##### 非公平锁和公平锁

ReetrantLock(boolean isfair)构造函数来确定使用公平还是非公平策略，

在实现上公平锁的**tryAcquire方法多了!hasQueuedPredecessors()来判断是否有前驱节点**，即有没有等待时间更久的线程。

```java
protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                // 公平锁这里多了判断是否有等待时间更久的线程
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
}
```



#### 使用示例

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class MyThread extends Thread {
    private Lock lock;
    public MyThread(String name, Lock lock) {
        super(name);
        this.lock = lock;
    }
    
    public void run () {
        lock.lock();
        try {
            System.out.println(Thread.currentThread() + " running");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } finally {
            lock.unlock();
        }
    }
}

public class AbstractQueuedSynchronizerDemo {
    public static void main(String[] args) throws InterruptedException {
        Lock lock = new ReentrantLock(true);
        
        MyThread t1 = new MyThread("t1", lock);        
        MyThread t2 = new MyThread("t2", lock);
        MyThread t3 = new MyThread("t3", lock);
        t1.start();
        t2.start();    
        t3.start();
    }
}
```

