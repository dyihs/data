[TOC]

### 一、 Docker Compose 配置

Compose是官方的开源项目，需要安装。Dockerfile 让程序在任何地方运行。

**(1) Compose重要概念**

- 服务service，容器。将每个容器关联起来，例如（web应用、redis、mysql、nginx），也就是将web服务需要的第三方组件关联，而每个组件是容器。

- 项目project，compose将每个容器关联起来，形成一个完整的项目。

**(2) 下载Docker Compose并授权**

在Docker官方文档中有下载地址，但是速度相当慢，使用下面的下载地址可以快速下载。

```bash
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.6.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 查看是否下载了docker-compose，
ls /usr/local/bin/
# 授权
chmod +x /usr/local/bin/docker-compose 
# 查看版本
docker-compose version
```

下载出现的[错误提示](#error)

**(3) 测试**

官网测试示例：https://docs.docker.com/compose/gettingstarted/ **推荐查看官方文档**

STEP1：创建一个`vim app.py` （web）

STEP2：创建Dockerfile构建 Web 镜像

STEP3：创建docker-compose.yml 文件，将镜像启动之后之间的网络进行互通这里有web、redis。

STEP4：启动Docker Compose，官方文档使用`docker compose up` 命令启动，但是在Ubuntu中会显示compose参数无效。使用 `docker-compose up` 命令可以正常启动。整个过程会先下载镜像层，然后执行Dockerfile

Docker Compose会将整个项目所依赖的镜像串联起来，包括启动镜像，镜像下载。在不使用Docker Compose时，需要每个镜像都要执行命令下载并启动以及挂载配置等。

**(4) 默认服务名**

使用 `docker ps` 查看启动的容器，会发现文件服务名称后面带有数字，也就是 -num，这是集群部署情况下会有多个服务器，所以会添加副本数量，因此使用 -num来标记。



### 二、docker-compose.yml 文件

docker-compose.yml 文件一共有三层，version、services、networks。

```yaml
version: "" 				# version: "3.0"
services:
  web:
    build: 				   # 当前目录 .
    ports:
      - ""             # 映射端口，"998080"
  db:                  # 数据存储选择
    image:              # 镜像选择，例如 mysql

networks:
  default:
    # Use a custom driver
    driver:             
```



### 三、<span color='red'>错误提示</span>

1、-bash: /usr/local/bin/docker-compose: Permission denied

如何是Ubuntu系统，切换为root用户，具体操作如下：

```bash
# 设置root用户密码
sudo passwd root

# 打开sshd_config
sudo vim /etc/ssh/sshd_config

# 编辑
# 1. 首先输入 /Authentication ,找到 Authentication 位置 
# 2. 然后英文输入下输入 i 之后进入编辑状态，找到PermitRootLogin 参数并设置为 yes，
# 3. 如果该参数有注释则取消注释 #。
# 4. 在该参数附近找到 PasswordAuthentication 参数并将其修改为 yes

# 保存并退出
# 按住Esc退出编辑状态，输入 wq ，保存文件并退出

# 重启
sudo service ssh restart
```

[解法方法链接](https://blog.csdn.net/thebestleo/article/details/123451471)

2、chmod: cannot access '/usr/local/bin/docker-compose': No such file or directory

```bash
chmod +x /usr/local/bin/docker-compose
```

