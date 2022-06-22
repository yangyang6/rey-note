![image-20220114165525086](/Users/yangli/Library/Application Support/typora-user-images/image-20220114165525086.png)



![image-20220114171842246](/Users/yangli/Library/Application Support/typora-user-images/image-20220114171842246.png)





双亲委派机制

![image-20220114182340260](/Users/yangli/Library/Application Support/typora-user-images/image-20220114182340260.png)





![image-20220120141525734](/Users/yangli/Library/Application Support/typora-user-images/image-20220120141525734.png)





对象整个生命周期大致可分为7个阶段：

* 创建阶段（Creation）
* 应用阶段（Using）
* 不可视阶段（Invisible）
* 不可到达阶段（Unreachable）
* 可收集阶段（Collected）
* 终结阶段（Finalized）
* 释放阶段（Free）





<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20220305110834897.png" alt="image-20220305110834897" style="zoom:50%;margin-left:-1px" />



对象与堆

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20220305114034476.png" alt="image-20220305114034476" style="zoom:50%;margin-left:-1px" />

Eden  -> S0 -> S1 -> Old





### 如何选择垃圾收集器

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20220307105053787.png" alt="image-20220307105053787" style="zoom:50%;margin-left:-1px" />



jmap：

Jstat：查看GC的状态

jstack：查看这个进程里面的线程状态  jstack 1212



