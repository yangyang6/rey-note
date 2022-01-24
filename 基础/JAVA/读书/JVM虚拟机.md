# 自动内存管理

### 运行时数据区域

![image-20220117103720920](/Users/yangli/Library/Application Support/typora-user-images/image-20220117103720920.png)

* 程序计数器

  当前线程所执行的字节码的行号指示器

  如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行虚拟机字节码指令的地址；如果正在执行的是本地（Native）方法，这个计数器值则应为空

* Java虚拟机栈

​		它的生命周期与线程相同

* 本地方法栈

​		本地方法栈与虚拟机栈所发挥的作用是非常相似的，其区别只是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的本地（Native）方法服务

* Java堆

​		Java堆是虚拟机所管理的内存中最大的一块，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象的实例

* 方法区

  它用于存储已被虚拟机加载的类型信息，常量、静态变量、即时编译器编译后的代码缓存等

* 运行时常量池

​		用于存放编译期生成的各种字面量与符号引用



### 对象的创建

只针对普通Java对象

### 对象的内存布局

对象在堆内存中的存储布局可以划分为三个部分：

* 对象头(Header)
* 实例数据（Instance Data）
* 对齐填充（Padding）：起着占位符的作用

![image-20220117150857773](/Users/yangli/Library/Application Support/typora-user-images/image-20220117150857773.png)

### 对象的访问定位

使用句柄访问的话，Java堆中将可能会划分出一块内存来作为句柄池

![image-20220117151725905](/Users/yangli/Library/Application Support/typora-user-images/image-20220117151725905.png)



通过直接指针访问对象

![image-20220117153118994](/Users/yangli/Library/Application Support/typora-user-images/image-20220117153118994.png)



首先要分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）

内存泄漏：可进一步通过工具查看泄漏对象到GC Roots的引用链，找到为什么垃圾回收器无法回收它们



### 垃圾收集

只有处于运行期间，我们才知道程序究竟会创建哪些对象，创建多少个对象，这部分内存的分配和回收是动态的

判断对象是否存活方法：

* 引用计数算法

​		在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一；当引用失效时，计数器就减一，任何时刻计数器为零的对象就是不可能再被使用的

​		缺点：不能处理循环引用问题等

* 可达性分析

​		基本思路是通过一些列称为GC Roots的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为“引用链”。如果GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的

![image-20220117172504965](/Users/yangli/Library/Application Support/typora-user-images/image-20220117172504965.png)



### 再谈引用

在JDK1.2版之后，Java对引用的概念进行了扩充

* 强引用

  类似Object obj = new Object()

* 软引用

​		用来描述一些还有用，但非必须的对象

* 弱引用

​		用来描述那些非必须对象，被弱引用关联的对象只能生存到下一次垃圾收集发生为止

* 虚引用

​		被称为"幽灵引用"或者“幻影引用”，为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知



### 生存还是死亡？

被GC Root标记了一次之后，还有一次生存机会，

```java
public class FinalizeEscapeTest {
    public static FinalizeEscapeTest SAVE_HOOK = null;

    public void is_alive() {
        System.out.println("I'm alive");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed!");
        FinalizeEscapeTest.SAVE_HOOK = this;
    }

    public static void main(String[] args) throws Exception {
        SAVE_HOOK = new FinalizeEscapeTest();

        //对象第一次成功拯救自己
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.is_alive();
        } else {
            System.out.println("no,I'm dead");
        }

        //下面这段代码与上面的完全相同，但是这次自救却失败了
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.is_alive();
        } else {
            System.out.println("no,I'm dead");
        }
    }
}
```

finalize确实被垃圾收集器触发过，并且在被收集前成功逃脱了。

结果是一次逃脱成功，一次失败。这是因为任何一个对象的finalize()方法都只会被系统自动调用一次，如果对象面临下一次回收，它的finalize()方法不会被再次执行，因此第二段代码的自救行动失败了

最后

finalize()方法不建议使用！



### 回收方法区

### 垃圾收集算法

本书介绍的是追踪式垃圾收集

##### 分代收集理论

* 弱分代假说（Weak Generational Hypothesis）：绝大多数对象都是朝生夕灭的
* 强分代假说（Strong Generational Hypothesis）：熬过越多次垃圾收集过程的对象就越难以消亡
* 跨代引用假说（Intergenerational Reference Hypothesis）：跨代引用相对于同代引用来说仅占极少数

因此有了Minor GC、Major GC、FULL GC这样的回收类型的划分；也才能够针对不同的区域安排与里面存储对象存亡特征相匹配的垃圾收集算法

* 标记-复制
* 标记-清除
* 标记-整理



部分收集(Partial GC)：指目标不是完整收集整个Java堆的垃圾收集：

* 新生代收集(Partial GC)：指目标只是新生代的垃圾收集
* 老年代收集(Major GC/Old GC)：指目标只是老年代的垃圾收集，目前只是CMS收集器会有单独收集老年代的行为。
* 混合收集(Mixed GC)：指目标是收集整个新生代以及部分老年代的垃圾收集。目前只有G1收集器会有这种行为

整堆收集(FULL GC)：收集整个Java堆和方法区的垃圾收集



* 标记-清除算法

​		缺点：（1）执行效率不稳定

​					（2）内存空间碎片化问题：会产生大量不连续的内存碎片

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20220117192125139.png" alt="image-20220117192125139" style="zoom:50%;margin-left:-1px" />

* 标记-复制算法

  缺点：（1）产生大量的内存间复制的开销

​		<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20220117192554941.png" alt="image-20220117192554941" style="zoom:50%;margin-left:-1px" />

Andrew Appel提出了一种更优化的半区复制分代策略，设计HotSpot虚拟机默认Eden和Survivor的大小比例是8:1

* 标记-整理算法

​	在对象存活率较高时就要进行较多的复制操作，效率将会降低。

​	是否移动回收后的存活对象是一项优缺点并存的风险决策，如果移动存活对象，尤其是在老年代这种每次回收都有大量对象存活区域，移动存活对象并更新所有引用这些对象的地方将会是一种极为负重的操作，而且这种对象移动必须全部暂停用户应用程序才能进行，也就是Stop The World

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20220117194411719.png" alt="image-20220117194411719" style="zoom:50%;margin-left:-1px" />

标记-清除 产生的内存碎片 只有通过类似“分区空闲分配链表”来解决内存分配问题

HotSpot虚拟机里面关注吞吐量的Parallel Scavenge收集器是基于标记-整理算法的，而关注延迟的CMS收集器则是基于标记-清除算法来的。

也有这种开始使用标记-清除算法，等内存碎片实在过多时则使用标记-整理算法来处理碎片



最新的ZGC和Shenandoah收集器使用读屏障（Read Barrier）技术实现了整理过程与用户线程并发执行

通常标记-清除算法也是需要停顿用户线程来标记、清理回收对象的，只是停顿时间相对而言要来的短而已



##### HotSpot的算法细节实现

##### 根节点枚举





##### 经典垃圾收集器

从JDK1.3开始，一直到现在最新的JDK13，HotSpot虚拟机开发团队为消除或者降低用户线程因垃圾收集而导致停顿的努力一直持续进行着，从Serial收集器到Parallel收集器，再到Concurrent Mark Sweep（Cms）和Garbage First（G1）收集器，最终至现在垃圾收集器的最前沿成果Shenandoah和ZGC等

* Serial收集器

​	一个单线程工作的收集器，在它进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束

![image-20220119172913624](/Users/yangli/Library/Application Support/typora-user-images/image-20220119172913624.png)

 它依然是HotSpot虚拟机运行在客户端模式下的默认新生代收集器，有着简单而高效

对于内存资源受限的环境，它是所有收集器里额外内存消耗（Memory FootPrint）最小的

Serial收集器对于运行在客户端模式下的虚拟机来说是一种很好的选择

* ParNew收集器

​	ParNew收集器实质上是Serial收集器的多线程<font style="color:red">并行</font>版本

![image-20220119174812101](/Users/yangli/Library/Application Support/typora-user-images/image-20220119174812101.png)

  解释并行与并发

  并行（Parallel）：并行描述的是多条垃圾收集器线程之间的关系，说明同一时间有多条这样的线程在协同工作，通常默认此时用户线程是处于等待状态

  并发（Concurrent）：并发描述的是垃圾收集器线程与用户线程之间的关系，说明同一时间垃圾收集器线程与用户线程都在运行。由于用户线程并未被冻结，所以程序仍然能响应服务请求，但由于垃圾收集器线程占用了一部分系统资源，此时应用程序的处理的吞吐量将受到一定的影响

* Parallel Scavenge收集器

​	是一款新生代收集器，基于标记-复制算法实现，也是能够并行收集的多线程收集器。

​	它的目标则是达到一个可控制的吞吐量（Throughput）

​    ![image-20220119182304779](/Users/yangli/Library/Application Support/typora-user-images/image-20220119182304779.png)

* Serial Old收集器

​		Serial Old是Serial收集器的老年代版本，它同样是一个单线程收集器，使用标记-整理算法

![image-20220120153734060](/Users/yangli/Library/Application Support/typora-user-images/image-20220120153734060.png)

* Parallel Old收集器

​		Parallel Old是Parallel Scavenge收集器的老年代版本，支持多线程并发收集，基于标记-整理算法

* CMS收集器（Concurrent Mark Sweep）

​		一种以获取最短回收停顿时间为目标的收集器

​		整个过程分为四个步骤：

​		1）**初始标记**（CMS Initial mark）：仅仅只是标记一下GC Roots能直接关联到的对象，速度很快。

​		2）并发标记（CMS concurrent mark）：从GC Roots的直接关联对象开始遍历整个对象图的过程

​		3）**重新标记**（CMS remark）：为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录

​		4)  并发清除（CMS concurrent sweep）：清理删除标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

​		其中初始标记、重新标记这两个步骤仍然需要“Stop The World”。 并发标记、并发清除耗时最长。

![image-20220120160327080](/Users/yangli/Library/Application Support/typora-user-images/image-20220120160327080.png)

​		有如下缺点：

​		1）CMS收集器对处理器资源非常敏感

​		2）CMS收集器无法处理“浮动垃圾”（Floating Garbage）

​		3）CMS是一款基于“标记-清除”算法实现的收集器，收集结束会有大量空间碎片产生。空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很多剩余空间，但就是无法找到足够大的连续空间来分配当前对象，而不得不提前触发一次FULL GC的情况

* Garbage First收集器

​		主要面向服务端应用的垃圾收集器，JDK9发布之日，G1宣布取代Parallel Scavenge加Parallel Old组合，称为服务端模式下的默认垃圾收集器

​		将内存回收的“行为”和“实现”进行分离

​		在G1收集器出现之前所有垃圾收集器 包括CMS在内，垃圾收集的目标范围要么是整个新生代（Minor GC）、要么就是整个老年代（Major GC），再要么就是整个Java堆（Full GC）。而G1跳出了这个樊笼，它可以面向堆内存任何部分来组成回收集（Collection Set，CSET）进行回收，衡量标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大，这就是G1收集器的Mixed GC模式

​		Region中还有一类特殊的Humongous区域，专门用来存储大对象。G1认为只要超过一个Region容量一半的对象即可判定为大对象

​		虽然G1仍然保留新生代和老年代的概念，但新生代和老年代不再是固定的了，它们是一系列区域（不需要连续）的动态集合，Region作为单次回收的最小单元，**优先处理回收价值收益最大的那些Region**，这也就是“Garbage First”名字的由来

![image-20220120175755490](/Users/yangli/Library/Application Support/typora-user-images/image-20220120175755490.png)

​		CMS收集器采用增量更新算法实现，而G1收集器则是通过原始快照（SATB）算法来实现

​		G1收集器的动作：

​		1）初始标记（Initial Marking）

​		2）并发标记（Concurrent Marking）

​		3）最终标记（Final Marking）

​		4）筛选回收（Live Data Counting and Evacuation）：涉及存活对象的移动，是必须暂停用户线程，由多条收集器线程并行完成的

​		G1收集器除了并发标记外，其余阶段也是要完全暂停用户线程		![image-20220121112729797](/Users/yangli/Library/Application Support/typora-user-images/image-20220121112729797.png)

​	可以由用户指定期望的停顿时间是G1收集器很强大的一个功能

​	从G1开始，最先进的垃圾收集器的设计导向都不约而同地变为追求能够应付应用的内存分配速率（Allocation Rate）

​	G1优点有很多，比如指定最大停顿时间、分Region的内存布局、按收益动态确定回收集这些创新性设计带来的红利

​	G1从整体来看是基于“标记-整理”算法来实现的收集器，但从局部（两个Region之间）上看又是基于“标记-复制”算法来实现

​	G1相对CMS，堆中的每个Region都必须有一份卡表

​	目前在小内存应用上CMS的表现大概率仍然会优于G1，而在大内存应用上G1则大多数发挥其优势

* ZGC收集器

​		JDK11中新加入的具有实验性质的低延迟垃圾收集器

​		ZGC和Shenandoah的目标是高度相似的，都希望在尽可能对吞吐量影响不大的前提下，实现在任意堆内存大小下都可以把垃圾收集的停顿时间限制下十毫秒以内的低延迟

​		在X64硬件平台下，ZGC的Region可以具有大、中、小三类容量：

​		1）小型Region（Small Region）：容量固定为2MB，用于放置小于256KB的小对象

​		2）中型Region（Medium Region）：容量固定为32MB，用于放置大于等于256KB但小于4MB的对象

​		3）大型Region（Large Region）：容量不固定，可以动态变化，但必须为2MB的整数倍，用于放置4MB或以上的大对象，每个大型Region中只会存放一个大对象

​		ZGC在作者看来成熟之后，将会成为服务端、大内存、低延迟应用的首选收集器有力竞争者

* Epsilon收集器





### 虚拟机及垃圾收集器日志

