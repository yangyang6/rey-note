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











### 无锁VS偏向锁VS轻量级锁VS重量级锁

感觉是一个逐渐升级的过程