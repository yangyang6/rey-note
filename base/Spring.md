### @Transaction 注解

默认是根据 uncheck exception 异常来回滚的



### Spring概述

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20210927112720721.png" alt="image-20210927112720721" style="zoom:30%;margin-left:-3px" />





### Spring Cloud全家桶

##### Eureka

满足CAP中的AP模型

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20210914105428032.png" alt="image-20210914105428032" style="zoom:50%;margin-left:-1px" />



<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20210914105402592.png" alt="image-20210914105402592" style="zoom:53%;margin-left:-2px" />

##### 源码

Euerka server的数据存储层是双层ConcurrentHashMap

```java
private final ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>> registry= new ConcurrentHashMap<String, Map<String, Lease<InstanceInfo>>>();
```

第一层 <spring.application.name , Map<>>

第二层 <instanceID , Lease<InstanceInfo>> 



### 织入

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20211008172600677.png" alt="image-20211008172600677" style="zoom:50%;margin-left:-2px" />

案例：

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20211008174351331.png" alt="image-20211008174351331" style="zoom:50%;margin-left:-1px" />





### Springboot自动装配

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20211009121232100.png" alt="image-20211009121232100" style="zoom:50%;margin-left:-1px" />

