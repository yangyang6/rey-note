# Docker

> Docker容器的本质：Namespace做隔离，Cgroups做限制，rootfs做文件系统



关于数据丢失情况

stop  不会丢

down  会丢

rm 会丢



### 虚拟机与容器：

其实容器就是花费更少的开销去节约资源，容器能够很快启动，不像虚拟机那样还要做开机操作，因为容器是调用一个内核，所以有安全隐患。

基于docker容器的镜像和虚拟机镜像的一个很大不同是容器是多层构成，它能在多个镜像之间共享和征用，如果某个已经被下载的容器镜像已经包含后面下载镜像的某些层，那么后面下载的镜像就无须再下载这些层（Layer）。



#### 例子：

busybox镜像，作为测试镜像，输出"hello world"

<code>docker run busybox echo "hello world"</code>

<code>docker run <image>:<tag></code>

Docker项目通过“容器镜像”，解决了应用打包这个根本性难题

容器其实是一种沙盒技术，沙盒就是能够像集装箱一样，把你的“应用”“装”起来的技术。

由于容器是寄宿在宿主机上面的，所以很多容器操作我们要是去考虑的，在生产环境中，没有人敢把运行在物理机上的Linux容器直接暴露在公网上。



### 网络

作为一个容器，它可以声明直接使用宿主机的网络栈（-net=host），即：不开启network namespace

```dockerfile
docker run -d -net=host --name nginx-host nginx
```

容器启动后，直接监听的就是宿主机的80端口

容器网络关系图：

<img src="https://static001.geekbang.org/resource/image/e0/66/e0d28e0371f93af619e91a86eda99a66.png" alt="img" style="zoom:30%;margin-left:-1px" />



覆盖网络关系图：

<img src="https://static001.geekbang.org/resource/image/b4/3d/b4387a992352109398a66d1dbe6e413d.png" alt="img" style="zoom:33%;margin-left:-1px" />