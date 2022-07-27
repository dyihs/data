[toc]

**2021年9月1**

### 1. Docker Swarm

`docker run` 容器启动不具有扩展容器，`docker service` 服务，具有扩缩容器并滚动执行。

**(1) 查看服务：**

```bash
# 创建一个可扩缩容的容器
docker service create -p 8080:80 --name mynginx nginx
# 查看服务
docker service ps mynginx
```

**(2) 动态扩缩容:**

假如有三个集群，动态扩展三个服务后，会随机将这三个服务分配到三个集群中，假如动态扩展10个，则会将10个容器动态分配，第一个集群是三个，第二个也有可能是三个，第三个就是四个了，分配给每个集群的数量是不确定的。

```bash
docker service update --replicas 3 mynginx
# 或者
docker service update scale mynginx-3
```

此时如果不想使用那么多开启的服务，也可以使用动态扩缩容方法关闭

```bash
docker service update --replicas 1 mynginx
# 或者
docker service update scale mynginx=1
```

注意⚠️：集群中工作节点无法使用命令，只有主节点可以

**(3) 移除服务**

```bash
docker service rm mynginx
# 查看是否移除服务
docker service ps
```

### 2. Docker Stack

docker-compose 单机部署项目。Docker Stack则是集群部署。

```bash
# 单机
docker-compose up -d wordpress.yaml
# 集群
docker stack deploy wordpress.yaml
```

### 3. Docker Secret

安全配置，密码配置，证书配置

```bash
docker secret --help
# 参数
create    创建一个证书
inspect   查看证书的信息
ls        查看有哪些证书
rm        删除证书
```

### 4. Docker Config

配置证书

```bash
docker config --help
# 参数
create    为证书创建一个配置文件
inspect   查看配置的信息
ls        查看有哪些配置文件
rm        删除配置文件
```

