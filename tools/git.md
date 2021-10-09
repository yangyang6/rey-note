### 回退commit

```
git reset --soft HEAD~1
```





CICD的时候  可以在commit的时候加一个暗号，去引导持续集成工具去部署

```
git commit -m "add auto_trigger_build"
```

