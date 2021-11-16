# MQ相关

### 消息队列中pull和push的理解

rocketmq、kafka都是pull模式的，Broker不再需要维护Consumer的状态，状态维护在Consumer



### Rocketmq Consumer from Where

全新的消费组才会使用到下面的消费策略

CONSUME_FROM_LAST_OFFSET

CONSUME_FROM_FIRST_OFFSET

CONSUME_FROM_TIMESTAMP

老的消费组都是按已经存储过的消费进度继续消费



