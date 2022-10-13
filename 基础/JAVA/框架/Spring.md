BeanFactory 和 ApplicationContext

BeanFactory 和 ApplicationContext都是接口，BeanFactory是Spring中最底层的接口，提供了最简单的容器的功能，只提供了<strong>实例化对象</strong>和<strong>获取对象</strong>的功能，ApplicationContext是Spring一个更高级的容器，提供了更多的有用的功能，如：获取Bean的详情信息（如定义、类型）、国际化功能、统一加载资源的功能、强大的事件机制、对web应用的支持等。

加载的区别：BeanFactory是延迟加载，ApplicationContext是在IOC启动的时候就一次创建所有的Bean


### AOP
aop的本质：源代码再加工
* 在哪里做再加工
    具体实现的时候，又分成2层：
        1.第一层叫做Joinpoint，表示底层AOP框架能够支持的位置，比如方法调用处，静态代码初始化块处等，不同的AOP框架能够支持的范围也不同，比如AspectJ可以支持到字段级别，而Spring AOP只支持方法调用处
        2.第二层叫做Pointcut，用于在Jointpoint限定范围内，选出用户感兴趣的点，Pointcut用类似正则表达式的方式来表示，比如within(com.cwnu.service.impl.*)
* 做什么样的再加工
    Spring AOP支持2种类型的再加工
        1.第一种叫做Advice，作用在方法调用层面，可以实现在方法调用之前（before）或之后（after）执行某些操作或者around整个方法
        2.第二种叫做Introduction，作用在类层面，可以让已经存在的类实现一个新的接口。
            比如已经存在类叫做OldClass，新接口叫做NewInterface，新接口的实现类叫做DefaultImplementation，Introduction简单说就是：
            1)在OldClass的类声明中加上implaments NewInterface
            2)把DefaultImplementation中的代码复制到OldClass中，作为NewInterface的实现
* 最后还有一个概念叫做Aspect，用于把上面提到的概念放到一起，Aspect本身没有意义，仅仅是作为一个容器存在