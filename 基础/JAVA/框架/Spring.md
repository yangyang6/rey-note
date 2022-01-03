BeanFactory 和 ApplicationContext

BeanFactory 和 ApplicationContext都是接口，BeanFactory是Spring中最底层的接口，提供了最简单的容器的功能，只提供了<strong>实例化对象</strong>和<strong>获取对象</strong>的功能，ApplicationContext是Spring一个更高级的容器，提供了更多的有用的功能，如：获取Bean的详情信息（如定义、类型）、国际化功能、统一加载资源的功能、强大的事件机制、对web应用的支持等。

加载的区别：BeanFactory是延迟加载，ApplicationContext是在IOC启动的时候就一次创建所有的Bean