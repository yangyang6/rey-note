# ReentrantLock

简单地讲就是：同一个线程对于已经获得到的锁，可以多次继续申请得到该锁的使用权

重入锁是一种颗粒度更小的锁，控制更方便，更强大



当然要有锁的基本特性：

* <strong>加锁</strong>
* <strong>解锁</strong>



ReentrantLock也不过是在锁的基本特性上加了一些<strong>"可重入"</strong>、<strong>"公平"</strong>、<strong>"非公平"</strong>等特性

默认ReentrantLock初始化的是一个<strong>“非公平锁”</strong>

```java
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() {
    sync = new NonfairSync();
}
```



公平锁和非公平锁的操作其实是获取锁步骤区分：

<strong>公平锁</strong>：

判断在此线程之前是否有其他线程在等待这个锁

```java
if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
  setExclusiveOwnerThread(current);
  return true;
}
```

<strong>非公平锁</strong>：

没有判断在此之前的排队线程，而是直接进行获锁操作，很有可能线程A比线程B先等待这把锁，但是B却获得到了锁，A继续等待（这种现象叫做：<strong>线程饥饿</strong>）



### 疑问

<strong>线程是什么时候排队的？</strong>

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

NofairSync和FairSync调用tryAcquire获取锁失败的时候，才会执行后续的acquireQueued



<strong>线程是如何排队的？</strong>

线程的两种等待模式：

<strong>SHARED：</strong>表示线程以共享的模式等待锁(如ReadLock)

<strong>EXCLUSIVE：</strong>表示线程以互斥的模式等待锁(如ReentrantLock)，互斥就是一把锁只能由一个线程持有，不能同时存在多个线程使用同一个锁



线程在队列中的状态枚举：

<strong>CANCELLED：</strong>值为1，表示线程的获锁请求已经“取消”

<strong>SIGNAL：</strong>值为-1，表示该线程一切都准备好了，就等待锁空间出来给我

<strong>CONDITION：</strong>值为-2，表示线程等待某一个条件(Condition)被满足

<strong>PROPAGATE：</strong>值为-3，当线程处在“SHARED”模式时，该字段才会被使用上



### 等待中的线程如何感知到锁空间并获得锁？

acquireQueued方法就是把放入队列中的这个线程不断进行“获锁”，直到它<strong>“成功或锁”</strong>或者<strong>“不再需要锁（如被中断）”</strong>

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
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



当线程被阻塞，也就没有循环什么事儿了（阻塞的线程将会让出CPU资源，该线程不会被CPU运行。）直到下次被唤醒，该线程不会继续进行循环体内的操作。





### 解锁

在其他线程在<strong>释放锁时“唤醒"</strong>被阻塞着的线程去”拿锁“





