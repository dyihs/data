### 安装Docker

[Docker文档](https://docs.docker.com/desktop/linux/install/ubuntu/)

首先更新软件包列表

```shell
sudo apt update
```

卸载旧版本的Docker

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
```

安装一些apt必备的软件包

```shell
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

添加官方Docker版本库的GPG密钥添加到系统中，显示为ok表示添加成功

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

也可以搜索密钥验证添加是否成功，搜索GPG密钥的后8个字符查看完整的GPG信息

```shell
sudo apt-key fingerprint 0DCECD77
# 显示结果
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

设置稳定版仓库

```shell
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) \
  stable"
```

安装Docker Engine-Community，先更新apt索引包

```shell
sudo apt-get update
# 安装 最新版本的Docker Engin
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
使用脚步自动安装
```shell
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```
### Docker 镜像常用命令

1. **启动镜像**

如果启动的镜像不在本地，则会自动下载并启动最新版本的镜像，如果没有找到则返回错误信息。

```shell
docker run mysql # 启动MySQL镜像
```

2. 搜索镜像

```shell
docker search mysql # 查找mysql镜像
```

3. **下载镜像**

```shell
docker pull mysql # 下载MySQL镜像
docker pull mysql:8.0 # 下载指定版本的镜像
```

下载镜像时是分层下载的，不同的镜像可以使用相同的镜像层，例如mysql镜像和nginx镜像有相同的镜像层，在先下载完mysql镜像后，下载nginx镜像时则不会下载相同的镜像，节省了资源。

4. **查看本地镜像命令**

```shell
docker images
# 带参数
-a，all # 查看所有镜像
```

5. **删除镜像**

删除的镜像也是分层删除，删除的是自身的不被其他镜像使用的分层镜像。

```shell
docker rmi -f # 删除所有部队镜像
# 通过镜像名称删除镜像
docker rmi -f mysql
# 也可以通过镜像的ID删除
docker rmi -f e733345bdf456
# 通过镜像ID删除多个镜像
docker rmi -f e733345bdf456 e733345bdf456 e733345bdf456
# 通过设置递归删除，$ 查询所有镜像并删除全部镜像
docker rmi -f $(docker images -aq)
```

### Docker 帮助命令

```shell
docker version # 查看版本信息
docker info    # 查看系统信息
docker --help  # 查看Docker命令
```

### Docker 容器命令

有了镜像才可以创建容器

1. **新建容器并启动**

```shell
docker run [可选参数] mysql # 启动mysql镜像并进入容器

# 可选参数
--name="Name" # 创建一个容器并选择一个名字用来区分容器
-d            # 后台方式运行
-it           # 使用交互方式运行，进入容器查看内容
-p            # 指定容器的端口 -p 8080：8080，主机端口映:射容器端口
-p            # 随机指定端口
```

2. 进入容器

```shell
# 进入容器后会改变服务名称
docker run it mysql /bin/bash
# 进入容器里面
docker attach 容器ID
```

3. 列出运行的容器

```shell
# 查看正在运行的容器
docker ps
# 查看正在运行和历史运行过的容器
docker ps -a
# 显示最近创建的容器个数
docker ps -n=1
# 显示容器的编号
docker ps -aq
```

4. 退出并停止容器

```shell
exit
```

5. 退出但是不停止容器：使用 Ctrl + P + Q 快捷键
  
6. 删除容器
  

```shell
docker rm 容器id  # 删除指定的容器但是不能删除正在运行的容器
docker rm -f $(docker ps -aq) # 删除所有容器
docker ps -a -q|xargs docker rm # 删除所有容器
```

7. 启动和停止容器

```shell
docker start 容器ID       # 启动容器
docker restart 容器ID     # 重启容器
docker stop 容器ID        # 停止正在运行的容器
docker skill 容器ID       # 强制停止当前容器
```

### 其他命令

1. 查看日志命令

```shell
docker logs --help
docker logs -tf --tail 容器ID
# 查看前10条日志
docker logs -tf --tail 10 容器ID  
```

2. 查看容器中进程信息

```shell
docker top 容器ID
```

3. 查看容器详细信息，包括挂载信息，配置，网络

```shell
docker inspect 容器ID
```

4. 进入当前正在运行的容器

```shell
docker exec -it 容器ID /bin/bash          # 进入容器开启一个新的终端，可以在里面操作
docker attach -it 容器ID /bin/bash        # 进入容器正在执行的终端，不会启动新的进程
```

5. 拷贝容器里面的文件到容器外部，例如将容器中home下的文件test.go拷贝到容器外部下home目录下也可以使用容器卷技术实现自动同步。

```shell
docker cp 容器ID:/home/test.go /home
```

6. 查看Docker内存、cpu使用情况

```shell
docker status
```

7. 添加参数限制内存

```shell
# 使用 -e 进行环境配置参数
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xms512m" elasticsearch:7.7.2
# 测试
curl localhost:9200
```
