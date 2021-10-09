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





### Redis过期策略如何实现的

##### 过期键的删除策略

一、惰性删除（执行所有的读写命令都会检查过期键是否过期【SET、LRANGE、SADD、HGET、KEYS等】，如果过期则删除过期键）

二、定期删除（定期循环扫描，删除）





### Redis实现分布式锁

常用的Redission框架实现分布式锁

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20210901132951369.png" alt="image-20210901132951369" style="zoom:60%;margin-left:-10px;" />





### JAVA实现Redis可重入锁

```java
public class RedisWithReentrantLock {
    private ThreadLocal<Map> lockers = new ThreadLocal<>();
    private Jedis jedis;

    public RedisWithReentrantLock(Jedis jedis) {
        this.jedis = jedis;
    }

    private boolean _lock(String key) {
        return jedis.set(key, "", "nx", "ex", 5L) != null;
    }

    private void _unlock(String key) {
        jedis.del(key);
    }

    private Map<String, Integer> currentLockers() {
        Map<String, Integer> refs = lockers.get();
        if (refs != null) {
            return refs;
        }
        lockers.set(new HashMap<>());
        return lockers.get();
    }

    public boolean lock(String key) {
        Map refs = currentLockers();
        Integer refCnt = refs.get(key);
        if (refCnt != null) {
            refs.put(key, refCnt + 1);
            return true;
        }
        boolean ok = this._lock(key);
        if (!ok) {
            return false;
        }
        refs.put(key, 1);
        return true;
    }

    public boolean unlock(String key) {
        Map refs = currentLockers();
        Integer refCnt = refs.get(key);
        if (refCnt == null) {
            return false;
        }
        refCnt -= 1;
        if (refCnt > 0) {
            refs.put(key, refCnt);
        } else {
            refs.remove(key);
            this._unlock(key);
        }
        return true;
    }

    public static void main(String[] args) {
        Jedis jedis = new Jedis();
        RedisWithReentrantLock redis = new RedisWithReentrantLock(jedis);
        System.out.println(redis.lock("codehole"));
        System.out.println(redis.lock("codehole"));
        System.out.println(redis.unlock("codehole"));
        System.out.println(redis.unlock("codehole"));
    }
}
```

