# Redis



### Redis常用命令

1.远程连接命令

```lua
redis-cli -h 192.168.32.34 -p 6379 -a 123456
```



### Redis集群

对于客户端请求的key，根据公式HASH_SLOT = CRC16（key） mod 16384，计算出映射到哪个分片上，然后Redis就会进入相应的节点进行操作



### 热Key的问题

通过hash去分key，把一个key拆分成多个key，分布到不同的节点，防止单节点过热。

也可以通过本地缓存简单的就用HashMap就可以了或者Guava cache或者Ehcache，用本地缓存来应对极热数据





### 常见的数据结构

* String
* Hash Tables
* Linked Lists
* Sets
* Sorted Sets

* bitmap：适用于海量数据的计算。特点是速度快，占用空间小

  1.可用于签到

  ​	SETBIT key offset value（设置或者清空key的value【字符串】在offset处的bit值【只能是0或1】）

  ​	如：

  ​		SETBIT 2018#yh 1 1

  ​		SETBIT 2018#yh 2 1

  ​	统计：

  ​		BITCOUNT 2018#yh