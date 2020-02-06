# drools相关笔记


## 定时器

<code>timer (cron: 0/2 * * * * ?)</code>    
<code>timer (cron: */2 * * * * ?)</code>  
每隔多少秒进行轮询， 这里两种配置方式都是一样的效果  


<code> timer (int: a b) </code>  
a表示延迟多少秒开始  
b表示隔多少秒轮询一次


## 左手定制