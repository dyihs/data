### 具名和匿名挂载

```shell
# 匿名挂载
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx
# 查看所有的 volume 的情况
docker volume ls
DRIVER    VOLUME NAME
local     0c77ad9d5d774fac49751eff3542af6e2751ab6d471c8e223228aca4d975b5d0
local     5eee9b66ec8bb466a3bb9e8117764ff5075fc3226ccba0a868dd66b6cdc27763
local     e9d03ec2473456f510b40e928b67f666c5d8db1df57954cbd8528342bd3bf4a2

# 这里发现匿名挂载的VOLUME NAME是没有指定名称的，使用 -v 写了容器内部的路径但是
# -没有写容器外部的路径

# 具名挂载
docker run -d -P --name nginx01 -v juming-nginx:/nginx nginx
# 查看具名挂载后VOLUME NAME
docker volume ls
DRIVER    VOLUME NAME
local     0c77ad9d5d774fac49751eff3542af6e2751ab6d471c8e223228aca4d975b5d0
local     5eee9b66ec8bb466a3bb9e8117764ff5075fc3226ccba0a868dd66b6cdc27763
local     e9d03ec2473456f510b40e928b67f666c5d8db1df57954cbd8528342bd3bf4a2
local     juming-nginx
```

### 查看挂载后的路径

容器挂载是通过 -v 具名卷名称:容器内路径，查看一下这个卷。所有的Docker容器内的卷，没有指定目录的情况下都是在 `/var/lib/docker/volumes/juming-nginx/_data`目录下。通过具名挂载方便找到挂载的卷，因此大多情况下使用具名挂载。

```shell
docker volume inspect juming-nginx
# 显示信息如下所示：
[
    {
        "CreatedAt": "2022-07-15T02:55:42Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]
```

区分匿名挂载和具名挂载

```shell
-v 容器内路径 # 匿名挂载
-v 卷名:容器外路径 # 具名挂载
-v /宿主机路径:容器内路径 # 指定路径挂载
```

挂载权限设置

```shell
# 通过 -v 容器内路径:ro/rw 改变读写权限
ro readonly # 只读
rw readwrite #  可读可写

# 一旦设置了容器权限，容器挂载到外部的内容就有了权限
ocker run -d -P --name nginx01 -v juming-nginx01:/etc/nginx:ro nginx
ocker run -d -P --name nginx02 -v juming-nginx02:/etc/nginx:rw nginx
```

使用 Dockerfile挂载

dockerfile 是用来构建docker 镜像文件的命令脚本，通过脚本一个一个命令生成一层一层的镜像，也就是每个命令就是一层一层的镜像。

```shell
# 创建一个dockerfile文件
mkdir dockerfile
# 编辑dockerfile，使用命令自动挂载镜像
vim dockerfile
# 编辑内容
FROM centos        # 以centos为基础
VOLUME ["volume01","volume02"]
CMD echo"------END------"
CMD /bin/bansh
```

注意⚠️ dockerfile文件中的挂载的文件名是 volume01 和 volume02，只有在这两个文件中的数据才会同步，不再该文件下的资源不会在两个容器之间进行同步。

执行脚步命令构建镜像

```shell
docker build -f /home/test-volume/dockerfile -t testvolume/centos:1.0 .
```

删除数据卷

```shell
docker rm -v [数据卷名称或者ID]
# 全部删除
docker volume prune
```

### 数据卷容器

多个mysql同步数据，使用 <mark>--volumes-from</mark> 挂载后数据会同步。

注意⚠️：只有使用volume挂载过的资源才会相互同步资源。

1. 创建两个容器并与其中一个挂载

```shell
# docker run -it --name [name] [image]
docker run -it --name docker01 centos:7.0
# 开启第二个容器并与第一个容器通过volumes-from挂载
docker run -it --name docker02 --volumes-from docker01 centos:7.0
```

2. 进入其中一个容器将创建一个测试文件

```shell
# docker attach [容器ID]
docker attach 7e4d4873fabd
# 创建一个文件
touch docker01
```

3. 进入到另一个容器查看有没有将文件同步过来

```shell
# docker attach [容器ID]
docker attach a4c8cba7b2dd
# 查看是否存在另一个容器中挂载的文件
ls -l 
# 创建一个空文件
touch docker02
# 退出当前容器 使用快捷：trl + p + q
# 进入到docker01容器
docker attach docker01
# 查看docker02容器中创建的文件是否存在
ls -l
```

如果所有的容器都与同一个容器挂载，就算将这个容器删除，其他容器之间也会同步数据。例如容器：docker2、docker3、docker4 ，都挂载到了容器docker1上，删除docker1容器，其他容器的创建和删除数据也是同步的。

### Dockerfile

创建镜像步骤：

1. 编写一个dockerfile文件
  
2. docker build 构建成为一个镜像
  
3. docker run 运行镜像
  
4. docker push 发布镜像
  

**1、dockerfile构建过程**

每个指令都必须是大写字母，执行从上到下顺序执行，每个指令都会创建提交一个新的镜像层然后提交。

![images](https://img0.baidu.com/it/u=1313280275,2971508472&fm=253&fmt=auto&app=138&f=PNG?w=764&h=500)

dockerfile是在发布项目时编写的命令，通过dockerfile生成项目所依赖的环境镜像，然后将镜像发布，获取镜像后就会无环境差别的部署项目，不存在环境导致项目部署失败或出现其他问题。Docker镜像已经成为了企业交付标准，必须掌握。

**2、dockerfile 的指令**

```dockerfile
FROM               # 基础镜像
MAINTAINER         # 镜像是谁写的，姓名+邮箱
RUN                # 镜像构建的时候需要运行的命令
ADD                # 添加其他内容，例如Tomcat镜像
WORKDIR            # 镜像的工作目录
VOLUME             # 挂载的目录
EXPOSE             # 保留端口配置
CMD                # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可以被替代
ENTRYPOINT         # 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD            # 当构建一个被继承的Dockerfile，就会运行ONBUILD的指令
COPY               # 类似ADD，将我们的文件拷贝到镜像中
ENV                # 构建的时候设置环境变量 
```

**3、编写一个Dockerfile**

DockerHub中大多镜像的基础镜像使用的是 `FROM scratch` ，例如Centos镜像。

```dockerfile
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20201113" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-11-13 00:00:00+00:00"

CMD ["/bin/bash"]
```

> 创建一个自己的centos，编写一个dockerfile，以centos为基础创建镜像

```dockerfile
FROM centos
MAINTAINER shiyd<syd@gmail.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "------end------"
CMD /bin/bash
```

注意⚠️构建dockerfile，注意后面加空格加英文句号

```bash
# docker build -f [dockerfile 文件] -t [镜像名称]:[版本] .
docker build -f mydockerfile-centos -t mycentos:1.0 .
```

在构建的过程中会复用docker中已经存在的镜像，然后一步一层的执行。也可以使用下面的命令查看其他镜像dockerfile文件构建的过程。

```bash
# docker history [容器ID]
docker history a7870fd478f4
```

> Dockerfile 构建Tomcat

```dockerfile
FROM centos
MAINTAINER shiyd<2986@qq.com>

COPY readme.txt /usr/local/readme.txt

# 添加压缩包到指定目录
ADD jdk-11.0.15_linux-aarch64_bin.deb /usr/local/
ADD apache-tomcat-9.0.64.tar.gz /usr/local/

# 添加其他命令
RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

# 环境配置
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool1.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.64
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.64
EVN PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

# 设置端口
EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.64/bin/startup.sh && tail -F /url/local/apache-tomcat-9.0.64/bin/logs/catalina.out
```

由于使用dockerfile作为文件名称，所以可以直接使用下面的命令进行构建，注意：下面的命令需要在dockerfile文件当前目录下使用，否则会出错。

```bash
docker build -f mytomcat .
```

### 发布自己的镜像

##### 1、发布到 docker hub

首先登录docker hub账号

```shell
docker login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username
```

登录成功后使用 `docker push` 命令将镜像发布到docker hub上。

在push镜像时需要添加自己版本信息和作者名字，当push出现错误时，多半时没有添加版本号（Tag），或者版本信息和作者信息错误，例如下面的错误：

```shell
[root@Arthur ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
mytomcat     latest    4a7662a923b0   About an hour ago   367MB
centos       latest    5d0da3dc9764   10 months ago       231MB

[root@Arthur ~]# docker push royshi/mytomcat:1.0
The push refers to repository [docker.io/royshi/mytomcat]
An image does not exist locally with the tag: royshi/mytomcat
```

解决方法，push带版本号的image：

```shell
[root@Arthur ~]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED             SIZE
mytomcat          latest    4a7662a923b0   About an hour ago   367MB
royshi/mytomcat   1.0       4a7662a923b0   About an hour ago   367MB
centos            latest    5d0da3dc9764   10 months ago       231MB

[root@Arthur ~]# docker push royshi/mytomcat:1.0
The push refers to repository [docker.io/royshi/mytomcat]
64020ff32f4c: Pushing [========>                     ]  11.39MB/66.29MB
13e6b396c7d6: Pushed
da549a94b57a: Pushed
a488bf80ae40: Pushing [=============>                ]  4.365MB/16.07MB
d30a7de76c1d: Pushing [====>                         ]  4.425MB/53.44MB
1e0e5ae046b5: Pushed
74ddd0ec08fa: Pushing [====>                         ]  21.32MB/231.3MB
```

### 练习过程中的错误

1、`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`

使用 `systemctl status docker` 命令查看docker是否启动，如果没有启动，则使用 `systemctl start docker` 命令启动docker。

2、`COPY failed: file not found in build context or excluded by .dockerignore: stat readme.txt: file does not exist`

这个错误出现的原因是执行命令时，资源不在当前dockerfile文件的当前目录下，如下命令中所示：

```dockerfile
COPY readme.txt /usr/local/readme.txt

# 添加压缩包到指定目录
ADD jdk-11.0.15_linux-aarch64_bin.deb /usr/local/
ADD apache-tomcat-9.0.64.tar.gz /usr/local/
```

readme.txt、jdk-11.0.15_linux-aarch64_bin.deb、apache-tomcat-9.0.64.tar.gz这三个文件必须和dockerfile文件同目录。

3、`Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist`

问题描述：当我在dockerfile中添加了 `RUN yum -y install vim` 后，构建到这一步之后出现了上面这个错误，在stackoverflow中发现有人也有类似的错误，[这是这个错误的连接](https://stackoverflow.com/questions/70963985/error-failed-to-download-metadata-for-repo-appstream-cannot-prepare-internal)。

解决方法：按照stackoverflow中其中一个方案，在 `RUN yum -y install vim` 命令之前添加命令：

```dockerfile
RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```
### Docker 网络

使用 `ip addr` 查看网络，分别有三个网卡所对应的地址：

- lo，本地回环地址
  
- eth0，云服务内网地址
  
- docker0，docker地址
  

<mark>**docker是如何处理容器网络访问的？**</mark>

启动并进入容器后，使用 `ip addr` 查看容器地址，会得到一个eth0@if262

每启动一个容器，docker就会给容器分配一个ip，只要安装了docker，就会有一个网卡docker0桥接模式，使用的技术veth-pair技术。启动容器后带来的网卡是一对veth，而veth-pair 就是一对的虚拟设备接口，他们都是成对出现的，一端连着协议，一端彼此相连，有了这个特性，veth-pair充当一个侨梁，连接各种虚拟网络设备的。

物理网卡与docker0相连，docker0通过veth-pair与各个容器相连。Docker中的所有网络接口都是虚拟的，虚拟转发效率高。通过`--link` 可以解决网络连通问题。

真实开发不建议使用 --link。在自定义网络下不适用docker0，docker0有局限性，不支持容器容器名连接访问。

### 自定义网络

使用 `docker network ls` 查看所有docker网络，docker0域名不能访问，--link可以打通连接。我们可以自定义一个网络

```shell
# --driver bridge
# --subnet 192.168.0.0/16
# --gateway 192.168.0.1
docker network create --driver bridge \
--subnet 192.168.0.0/16 \
--gateway 192.168.0.1 \
mynet

# 查看网络
docker network ls
```

### 网络连通

```bash
[root@Arthur ~]# docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

connect 命令参数就是连接网络的参数，将容器连接到网络。查看connect命令参数如何使用：

```bash
[root@Arthur ~]# docker network connect --help

Usage:  docker network connect [OPTIONS] NETWORK CONTAINER

Connect a container to a network # 连接一个容器到网络

Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
[root@Arthur ~]#
```
