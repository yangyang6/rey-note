### Redis缓存过期策略

设置最大maxmemory，设置了maxmemory就要设置maxmemory-policy

maxmemory默认的过期策略是volatile-lru（只对设置了过期时间的key进行LRU）

redis.conf

```js
maxmemory 1024mb 
```



### 删除策略

##### 定时删除

##### 惰性删除

##### 主动删除



maxmemory-policy六种方式：

* Volatile-lru：只对设置了过期时间的key进行LRU
* Allkeys-lru：删除lru算法的key
* volatile-random：随机删除即将过期的key
* allkeys-random：随机删除
* volatile-ttl：删除即将过期的
* Noeviction：永不过期，返回错误





### 通信协议及事件处理机制

##### 通信协议

Redis是单进程单线程的

应用系统和Redis通过Redis协议（RESP）进行交互

##### 请求响应模式

Redis协议位于TCP层之上，即客户端和Redis实例保持双工的连接

##### 规范格式（redis-cli）RESP

1、间隔符号 在linux下 \r\n 在windows下是\n

2、简单字符串simple string，以”+“加号开头

3、错误Errors，以“-”减号开头

4、整数型Integer，以“:”冒号开头

5、大字符串类型Bulk Strings，以“$”美元符号开头，长度限制512M

6、数组类型Arrays，以“*”星号开头