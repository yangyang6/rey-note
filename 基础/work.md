

### 杀掉所有java进程

```shell
ps -ef | grep java | grep -v grep | awk '{print $2}' | xargs kill -9
```

