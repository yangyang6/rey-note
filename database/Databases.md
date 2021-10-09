

Mac 上直接启动mongo

```shell
brew services start mongodb-community@4.4
```



# Mysql

一条sql语句的生命周期

<img src="https://static001.geekbang.org/resource/image/0d/d9/0d2070e8f84c4801adbfa03bda1f98d9.png?" alt="img" style="zoom:40%;margin-left:-10px;" />





更新语句

redo log（server引擎持有的日志 - 物理日志）、binlog（归档日志，Server层的日志 - 逻辑日志）

redo log固定大小，从头开始写，写到末尾就又回头开始循环写

<img src="https://static001.geekbang.org/resource/image/16/a7/16a7950217b3f0f4ed02db5db59562a7.png?" alt="img" style="zoom:40%;margin-left:-10px;" />



所以有了checkpoint 和 writepos 之后就有了crash-safe的概念了

MyIsam不支持事务



当数据库有多个事务同时执行的时候，就可能出现脏读

对事务的理解，区分<strong>读未提交</strong>与<strong>读提交</strong>这两个主要是在事务未提交/提交时  它所做的事儿能不能被其他事务看到

串行化指读、写都加锁