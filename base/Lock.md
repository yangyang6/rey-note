# Lock

关于Java的锁，总结

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/7f749fc8.png)



![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/c8703cd9.png)



![image-20200730173109646](/Users/yangli/Library/Application Support/typora-user-images/image-20200730173109646.png)



CAS（Compare And Swap），是一种无锁算法，在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步，java.util.concurrent包中的原子类就是通过CAS来实现乐观锁。



CAS算法涉及三个操作数：

* 需要读写的内存值V
* 进行比较的值A
* 要写入的新值B



CAS存在三大问题：

* ABA问题（加入版本号）
* 循环时间开销大
* 只能保证一个共享变量的原子操作（多个变量操作时，CAS是无法保持原子性的，Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里面来进行CAS操作）



### 自旋锁

阻塞或唤醒一个Java线程需要操作系统切换CPU状态来完成，这种状态转换需要耗费处理器时间。如果同步代码块中的内容过于简单，状态转换消耗的时间有可能比用户代码执行的时间还要长



<strong>AQS原理</strong>

理解不深

<strong>AQS主要是采用state，通过对state的CAS判断来获取锁和解锁，并且存在等待队列和条件等待队列来park相关线程之后入队等待，有公平和非公平两者模式来唤醒等待的线程</strong>



# 共享锁

读锁，当上锁之后，另一个线程中可以读，不可以修改



# 排他锁

上锁之后，另外一个线程不能读和修改







### 无锁VS偏向锁VS轻量级锁VS重量级锁

感觉是一个逐渐升级的过程







# 自旋锁(SpinLock)

是指当一个线程在获取锁的时候，如果锁被其他线程获取，那么该线程将循环等待，然后不断判断锁是否能够被成功获取，直到获取到锁才推出循环



# 优点

自旋锁不会使线程状态发生切换，一直处于用户态，即线程一直是active，不会使线程进入阻塞状态，减少了不必要的上下文切换，执行速度快。



非自旋锁在获取不到锁的时候会进入阻塞状态，从而进入内核态，当获取到锁的时候需要从内核态恢复，需要线程上下文切换。 （线程被阻塞后便进入内核（Linux）调度状态，这个会导致系统在用户态与内核态之间来回切换，严重影响锁的性能）









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





