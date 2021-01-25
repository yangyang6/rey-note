# Docker





# 日常运维

### 日志查看

docker Exited 后 重启

```dockerfile
docker start containerID 
```



查看错误日志

```dockerfile
docker logs -f --tail 2000  api-operation-app |grep "employee/login"
```



查看一段时期里面的日志

```dockerfile
docker logs --since 2021-01-18T10:48:00 --until 2021-01-18T10:49:00 service-profile
```



