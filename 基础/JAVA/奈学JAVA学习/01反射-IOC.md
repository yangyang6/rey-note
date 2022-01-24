# IOC

控制反转是一种设计思想，并非实际的技术，最核心的思想就是将预先设计的对象实例创建的控制权交给程序（IOC容器）

注：IOC容器大家可以简单认为是一种KV的Map集合



AbstractApplicationContext -> BeanFactory getBean()  -> AbstractBeanFactory doGetBean()

![image-20220110010611304](/Users/yangli/Library/Application Support/typora-user-images/image-20220110010611304.png)



手写IOC的实现





DI依赖注入：

* 构造器注入
* Setter方法注入