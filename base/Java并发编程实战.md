# 简介

>  CPU调度的基本单位是线程。线程是操作系统能够进行运算调度的最小单位



```java
//@NotThreadSafe
public class UnsafeSequence {
    private int value;
    
    public int getNext() {
        return value++;
    }
}
```

在UnsafeSequence类中说明一种常见的并发安全问题，称为<strong>竞态条件（Race Condition）</strong>



在多线程程序中，当线程调度器临时挂起活跃线程并转而运行另一个线程时，就会频繁地出现上下文切换（Context Switch），这种操作将带来极大的开销。





# 线程安全性

要编写线程安全的代码，其核心在于要对状态访问操作进行管理，特别是对共享的（Shared）和可变的（Mutable）状态的访问



> Java中主要同步机制是关键字synchronized，它提供了一种独占的加锁方式，但“同步”这个术语还包括volatile类型的变量，显式锁（Explicit Lock），以及原子变量



> 无状态对象一定是线程安全的。（XXX类是无状态的，它既不包含任何域，也不包含任何对其他类中域的引用）



一旦加入了count++，这是一个“读取 - 修改 - 写入”的操作序列

> 在并发编程中，这种由于不恰当的执行时序而出现不正确的结果是一种非常严重的，它有一种正式的名字：竞态条件（Race Condition）。最常见的竞态条件类型就是“先检查后执行”（Check - Then - Act）操作，即通过一个可能失效的观测结果来决定下一步的动作。



在java.util.concurrent.atomic包中包含了一些原子变量类，用于实现在数值和对象引用上的原子状态转换。

> AtomicLong是一种替代long类型整数的线程安全类，类似地，AtomicReference是一种替代对象引用的线程安全类。

> 要保证状态的一致性，就需要在单个原子操作中更新所有相关的状态变量



### 内置锁（Intrinsic Lock）或监视器（Monitor Lock）

Java提供了一种内置的锁机制来支持原子性：同步代码块（Synchronized Block）

Java的内置锁相当于一种互斥体（或互斥锁），这意味着最多只有一个线程能持有这种锁

并发环境中的原子性与事务应用程序中的原子性有着相同的含义  —— 一组语句作为不可分割的单位被执行。任何一个执行同步代码块的线程，都不可能看到其他线程正在执行由同一个锁保护的同步代码块



### 重入

由于内置锁是可重入的，因此如果某个线程试图获得一个已经由它所持有的锁，那么这个请求就会成功。重入的一种实现方法是，为每个锁关联一个获取计数值和一个所有者线程，当计数值为0时，这个锁就被认为是没有被任何线程持有。当线程请求一个未被持有的锁时，JVM将记下锁的持有者，并且将获取计数值置为1。

```java
public class LoggingWidget extends Widget{
    @Override
    public synchronized void doSomething() {
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();
    }
}

class Widget{
    public synchronized void doSomething(){
    }
}
```



如果没有重入锁，上面的程序将会死锁（调用super.doSomething时将无法获得Widget上的锁，因为这个锁已经被持有）



> 虽然synchronized方法可以确保单个操作的原子性，但如果要把多个操作合并为一个复合操作，还是需要额外的加锁机制，此外，将每个方法都作为同步方法还可能导致活跃性问题（Liveness）或性能问题（Performance）



> Java内存模型要求，变量的读取操作和写入操作都必须是原子操作

> 加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，所有执行读操作或者写操作的线程都必须在同一个锁上同步

> voliatile变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值
>
> 加锁机制既可以确保可见性又可以确保原子性，而voliatile变量只能确保可见性



### ThreadLocal类

ThreadLocal对象通常用于防止对可变的单实例变量（Singleton）或全局变量进行共享





### 不变性

如果某个对象在被创建后其状态就不能被修改，那么这个对象就称为不可变对象。线程安全性是不可变对象的固有属性之一。



> 不可变对象即对象一旦被创建它的状态（对象的数据、也即对象的属性值）就不能改变，任何对它的改变都应该产生一个新的对象

