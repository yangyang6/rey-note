# Kong

### 一、概述

为什么需要网关 ？   网关是什么？  

### 二、Docker安装

> 参考：官网

##### 1.create a docker network

docker network create kong-net

##### 2.安装postgres数据库，并设置账户和信息

进入postgres

```dockerfile
docker run -d --name kong-database \
			--network=kong-net \
			-p 5432:5432 \
			-e "POSTGRES_USER=postgres" \
			-e "POSTGRES_PASSWORD=postgres" \
			-e "POSTGRES_DB=postgres" \
			postgres:9.6
```

```dockerfile
docker exec -it kong-database /bin/bash
```



\#切换账户

\> su postgres 

\#进入命令

\> psql



\l 列出所有的数据库

\d 列出所有的表



创建了数据库账户kong，和密码kong ，创建了数据库kong

```sql
create user kong with password 'kong';
create database kong owner kong;
```



创建了数据库账户konga，和密码konga ，创建了数据库konga

```sql
create user konga with password 'konga';
create database konga owner konga;
```



退出postgres 命令: \q

退出docker命令 exit



##### 3.安装kong

prepare your databases，初始化数据库（迁移数据），当kong的版本大于0.15时使用bootstrap

```dockerfile
docker run --rm \
		--network=kong-net \
		-e "KONG_DATABASE=postgres" \
		-e "KONG_PG_HOST=kong-database" \
		-e "KONG_PG_PORT=5432" \
		-e "KONG_PG_USER=kong" \
		-e "KONG_PG_PASSWORD=kong" \
		-e "KONG_PG_DATABASE=kong" \
		kong:latest kong migrations bootstrap
```



start kong：

```dockerfile
docker run -d --name kong \
		--network=kong-net \
		-e "KONG_DATABASE=postgres" \
		-e "KONG_PG_HOST=kong-database" \
		-e "KONG_PG_PASSWORD=kong" \
		-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
		-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
		-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
		-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
		-e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
		-p 80:8000 \
		-p 443:8443 \
		-p 8001:8001 \
		-p 8444:8444 \
		kong:latest
```

验证有无安装成功

curl -XGET http://localhost:8001



##### 4.安装konga

拉konga镜像：

docker pull pantsel/konga:latest



查看postgres docker容器的gateway 

docker inspect 61ebcbad38d6(容器id) | grep Gateway



init konga：

```dockerfile
docker run --rm pantsel/konga:latest -c prepare -a postgres -u postgresql://konga:konga@172.18.0.1:5432/konga
```



run konga

```dockerfile
docker run -d --name konga \
				--network=kong-net \
				-p 1337:1337 \
				-e "DB_ADAPTER=postgres" \
				-e "DB_HOST=172.18.0.1" \
				-e "DB_PORT=5432" \
				-e "DB_USER=konga" \
				-e "DB_PASSWORD=konga" \
				-e "DB_DATABASE=konga" \
				-e "KONGA_HOOK_TIMEOUT=120000" \
				-e "NODE_ENV=production" \
				pantsel/konga
```



维护konga用户信息

admin/12345678



### Konga的介绍

##### 演示 创建upstream，service，route



##### UPSTREAM

upstream就是一个虚拟的服务，可以用于配置多个target用来实现负载均衡。
当我们部署集群时，一个单独的地址不足以满足我们的时候，我们可以使用Kong的upstream来进行设置。
在service中的host可指定为upstream对象，upstream添加多个target来实现负债均衡。
注意:service的host指的就是upstram的name



##### SERVICE

服务实体，顾名思义，是您自己的上游服务的抽象,这个是官方文档的原话。其实理解起来就是就是我们自己定义的上游服务(upstraem),
**通过Kong匹配到相应的请求要转发的地方**,Service 可以与下面的Route进行关联,一个Service可以有很多Route,
匹配到的Route就会转发到Service中。
新增service后，kong会自动分配一个id值，该id为service的唯一标识。可用于后期修改、查看以及绑定route、upstream。
service 是抽象层面的服务，他可以直接映射到一个物理服务(target)(host 指向 ip + port)，也可以指向一个upstream来做到负载均衡。

|      |                                                 |
| ---- | ----------------------------------------------- |
| post | /certificates/{certificate name or id}/services |



##### ROUTE

首先有一个大概的理念就是，route 是路由的抽象，他负责将实际的 request 映射到 service。
实际上是通过定义一些规则来匹配客户端的请求，每个路由都会关联一个service，并且service可以关联多个route
当匹配到客户端的请求时，每个请求都会被代理到其配置的Service中。
Route作为客户端的入口，通过将Route和Service的松耦合，可以通过hosts path等规则的配置，最终让请求到不同的Service中。
例如，我们规定api.example.com 和 api.service.com的登录请求都能够代理到10.11.11.11:8000端口上，
那我们可以通过hosts和path来路由
首先，创建一个Service s1，其相应的host和port以及协议为http://10.11.11.11:8000
然后，创建一个Route，关联的Service为s1，其hosts为api.service.com, api.example.com,path为login
最后，将域名api.example.com和api.service.com的请求转到到我们的Kong集群上，
也就是我们上面一节中通过Nginx配置的请求地址
那么，当我们请求api.example.com/login和api.service.com/login时，
其通过Route匹配，然后转发到Service，最终将会请求我们自己的服务。

|      |                                       |
| ---- | ------------------------------------- |
| POST | /services/{service name or id}/routes |



##### TARGET

target 就是在upstream进行负载均衡的终端，当我们部署集群时，需要将每个节点作为一个target，
并设置负载的权重， 当然也可以通过upstream的设置对target进行健康检查。
当我们使用upstream时，整个路线是 Route >> Service >> Upstream >> Target
target 代表了一个物理服务，是 ip + port 的抽象；



> upstream 和 target ：   1 对 n
> service 和 upstream ： 1 对 1 或 1 对 0 （service 也可以直接指向具体的 target，相当于不做负载均衡）
> service 和 route：         1 对 n



cwnu.conf

```nginx
upstream cwnu_nodes{
	server 192.168.4.3:8899 weight=100
  server 192.168.4.3:8877 weight=100
}

server{
	listen 80;
	server_name test.app.com
	
	location /path{
		proxy_pass http://geek-ups;
	}
}
```



##### 丰富的插件

##### 自定义插件

middleman



