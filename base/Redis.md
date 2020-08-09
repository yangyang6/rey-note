# Redis

### Redis集群

对于客户端请求的key，根据公式HASH_SLOT = CRC16（key） mod 16384，计算出映射到哪个分片上，然后Redis就会进入相应的节点进行操作



### 热Key的问题

通过hash去分key，把一个key拆分成多个key，分布到不同的节点，防止单节点过热。

也可以通过本地缓存简单的就用HashMap就可以了或者Guava cache或者Ehcache，用本地缓存来应对极热数据



