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

