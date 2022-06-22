https://blog.csdn.net/Eoneanyna/article/details/112304809



mymon

```shell
docker run -itd --name mymon --network ceph-network --ip 172.20.0.10 -e MON_NAME=mymon -e MON_IP=172.20.0.10 -v /etc/ceph:/etc/ceph ceph/mon
```





启动osd0

```shell
docker exec mymon ceph osd create

docker run -itd --name osd0 --network ceph-network -e CLUSTER=ceph -e WEIGHT=1.0 -e MON_NAME=mymon -e MON_IP=172.20.0.10 -v /etc/ceph:/etc/ceph -v /var/lib/ceph/osd/0:/var/lib/ceph/osd/ceph-0 ceph/osd
```



