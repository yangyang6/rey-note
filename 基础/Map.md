# Map的相关总结



### HashMap

快速失败(fast-fail)

java.util下面的集合类都是基于此机制（包括HashTable）





### CurrentHashMap

安全失败(fail-safe)

java.util.concurrent包下的容器都是基于此机制



扩容机制：

TREEIFY_THREHOLD = 8是升级 成红黑树

UNTREEIFY_THREHOLD = 6是退化 成链表