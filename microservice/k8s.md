### 基操

查看deployment的信息

<code>kubectl get deploy --namespace=default</code>



删除默认命名空间的里面的deployment

<code>kubectl delete deploy nginx-deployment --namespace=default</code>



查看存在的工作空间(Namespace)

<code>kubectl get ns</code>



kube-system是k8s项目预留的系统pod的工作空间(Namespace)

<code>kubectl get pods -n kube-system</code>

![image-20200722163727779](/Users/yangli/Library/Application Support/typora-user-images/image-20200722163727779.png)

新手小白很多时候不懂怎么定位，之前pod一直启动报pending，找了很多资料才是网络组件没有安装，部署了网络组件就可以了：

<code>kubectl apply -f https://git.io/weave-kube-1.6</code>



我这里部署了一个单台master的k8s环境，由于默认情况下Master节点是不允许运行用户的pod的，k8s通过Taint/Toleration机制

![image-20200722170554715](/Users/yangli/Library/Application Support/typora-user-images/image-20200722170554715.png)





k8s对现有的容器进行水平扩展：

![image-20200722171644175](/Users/yangli/Library/Application Support/typora-user-images/image-20200722171644175.png)

![image-20200722171707880](/Users/yangli/Library/Application Support/typora-user-images/image-20200722171707880.png)

<code>kubectl scale deployment nginx-deployment --replicas=2</code>

水平扩展后：

![image-20200722171730081](/Users/yangli/Library/Application Support/typora-user-images/image-20200722171730081.png)





查看历史版本

<code>kubectl rollout history deployment/nginx-deployment</code>



运行一个

<code>kubectl run -i --tty --image busybox:1.28.4 dns-test --restart=Never --rm /bin/sh</code>





# Service

k8s 之所以需要service：

* pod的ip是不固定的
* 一组pod会有负载均衡的需求



Iptables -> IPVS （“将重要操作放到内核态”是提高性能的重要手段）

ClusterIP模式的Service为你提供就是一个Pod的稳定IP地址，即VIP，并且这里Pod和Service关系是可以过Label确定的

Headless Service为你提供的则是一个Pod的稳定的DNS名字，并且这个名字是可以通过Pod名字和Service名字拼接起来的



在使用k8s的service，如何从外部（k8s集群之外），访问到k8s里创建service？

SNAT操作？

当然k8s能够为service分配共有IP地址，externalIPs

# StatefulSet

StatefulSete的核心功能就是通过某种方式记录这些状态，然后在Pod被重新创建时，能够为新的的Pod恢复这些状态



![image-20200722204144456](/Users/yangli/Library/Application Support/typora-user-images/image-20200722204144456.png)

Headless Service不需要分配一个VIP，而是可以直接以DNS记录的方式解析出被代理Pod的IP地址，当你按照这样的方式创建了一个Headless Service之后，它所代理的所有Pod的IP地址，都会被绑定一个这样格式的DNS记录，如下所示：

<code><pod-name>.<svc-name>.<namespace>.svc.cluster.local</code>



### 存储状态

k8s项目引入一组叫做PVC和PV的API对象，大大降低了用户声明和使用持久化volumn的门槛

PVC和PV的设计实际上类似于“接口”和“实现”的思想，PVC提供了对某种持久化存储的描述，但不提供具体的实现；而这个持久化存储的实现部分则由PV负责完成。k8s里面有一个专门维护持久化存储的控制器（Volumn Controller），它维护者多个控制循环，其中有个循环就是撮合PVC和PV之间的角色，它叫做PersistentVolumeController

k8s为我们提供了可以自动创建PV的机制，即Dynamic Provisioning，而它的核心在于一个名为StorageClass的API对象



### 总结

StatefulSet的控制器直接管理的是Pod

其次，k8s通过Headless Service为这些有编号的Pod，在DNS服务器中生成带有同样编号的DNS记录

最后，StatefulSet还为每一个Pod分配并创建一个同样编号的PVC





# 声明式API

一个API对象在Etcd里的完整资源路径，是由Group（API组）、Version（API版本）、Resource（API资源类型）三个部分组成

具体的树形结构：

<img src="/Users/yangli/Library/Application Support/typora-user-images/image-20201112113900434.png" alt="image-20201112113900434" style="zoom:50%;margin-left:0px" />





# RBAC 角色控制

### Role：角色

### Subject：被作用者

### RoleBinding：绑定了被作用者与角色之间的关系

在k8s中已经内置了很多为系统保留的clusterRole，可以通过kubetcl get clusterroles查看



# Ingress

全局的、为了代理不同后端service而设置的负载均衡服务



感觉类似于nginx的功能





# NetworkPolicy

定义网络策略，入口与出口





# 	资源调度

在k8s中，cpu这种资源是“可压缩资源”，当可压缩资源不足时，Pod只会“饥饿”，不会退出

内存这样的资源，是“不可压缩资源”，当不可压缩资源不足时，Pod就会因为OOM被内核杀掉

500m的设置是指500millicpu，也就是指0.5个CPU



建议将DaemonSet的Pod都设置为Guaranteed的Qos类型