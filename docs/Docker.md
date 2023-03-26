# 概念

## 是什么

解决了运行环境和配置问题的软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。



## 三要素

镜像、容器、仓库

Docker 本身是一个容器运行载体或称之为管理引擎。

我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是image镜像文件。

只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。

image文件可以看作是容器的模板。

Docker 根据 image 文件生成容器的实例。

同一个 image 文件，可以生成多个同时运行的容器实例。 

**镜像文件**：image 文件生成的容器实例，本身也是一个文件，称为镜像文件。

**容器实例**：一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器

**仓库**：就是放一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候再从仓库中拉下来就可以了。 

## 镜像加载原理

UnionFS(联合文件系统)：Union 文件系统 (UnionFS)是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。 Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像。

**docker 的镜像实际上由一层一层的文件系统组成，这种层级的文件系统 UnionFS。**bootfs(boot file system)主要包含 bootloader 和 kernel, bootloader 主要是引导加载 kernel, Linux 刚启动时会加载 bootfs 文件系统，在 Docker 镜像的最底层是引导文件系统 bootfs。

rootfs (root file system) ，在 bootfs 之上。包含 的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs 就是各种不同的 操作系统发行版，比如 Ubuntu，Centos 等。

# 使用

启动docker

通过 launchctl 查看 docker server, 记住docker server 名

```shell
> launchctl list | grep docker
111117   0       com.docker.docker.2388
```

然后关闭和启动它。

```shell
launchctl stop com.docker.docker.2388 
launchctl start com.docker.docker.2388
```

// 上面这个不太好使



```shell
# 查看Docker版本
docker -v
docker --version
docker version

# 查看Docker运行相关信息
docker info

# 查看docker的镜像文件
docker images

# 创建容器
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# OPTIONS:
# -d：后台运行容器，并返回容器ID; 不需要
# -i：以交互模式运行容器，通常与-t同时使用; interactive
# -t：为容器重新分配一个伪输入终端; tty terminal
# -p：指定端口映射，格式为："主机端口:容器端口"
# -P：随机端口映射，容器内部端口随机映射到主机的端口
# -v，--volume：指定卷的映射，格式为："主机卷:容器卷"
# --name="new name"：为容器指定一个名称; 不写就随机分配
# --dns 8.8.8.8：指定容器使用的DNS服务器，默认和宿主一致
eg：
docker run -it 镜像名:版本号 /bin/bash
# 版本号要是latest可以省略

# 容器的启动关闭重启
docker start 容器名或id [CONTAINER...]
docker stop CONTAINER [CONTAINER...]
docker restart CONTAINER [CONTAINER...]

# 查看启动的容器
docker ps

# 进入容器
docker exec –it 容器名(或id) /bin/bash
# 使用exec之后用exit退出不会停止容器

# 退出
exit # 容器会停止
ctrl+p+q # 容器不会停止

# 删除已经停止的容器
docker rm 容器名或id
```

### 后台守护式启动

比如：redis

```shell
docker run -d redis
```

如果后台启动的容器没有进程的话，会自杀，比如centos这种

但是redis和mysql这种会一直存在进程，不会自杀

### 查看日志

```shell
docker logs 容器id
```

### 查看容器内部细节

```shell
docker inspect 容器id
```

显示的是一串json，有容器的基本数据

### 从容器上拷贝文件

```shell
docker cp 容器id:容器内路径 目的主机路径
```

### 导入和导出容器

```shell
# export 导出容器的内容留作为一个 tar 归档文件[对应 import 命令]
docker export 容器ID > 文件名.tar
# import 从 tar 包中的内容创建一个新的文件系统再导入为镜像[对应 export]
cat 文件名.tar | docker import -镜像用户/镜像名:镜像版本号
```

### commit操作

docker commit 提交容器副本使之成为一个新的镜像

```shell
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名(比如版本号)]
```



## 容器卷

Docker 挂载主机目录访问如果出现 cannot open directory .: Permission denied 解决办法:在挂载目录后多加一个`--privileged=true` 参数即可

## 挂载数据或配置文件

```shell
1 在docker hub搜索redis镜像
docker search redis

2 拉取redis镜像到本地
docker pull redis:6.0.10

3 修改需要自定义的配置(docker-redis默认没有配置文件，自己在宿主机建立后挂载映射)
创建并修改/usr/local/redis/redis.conf
# bind 127.0.0.1 开启远程权限
appendonly yes 开启aof持久化
密码之类的
daemonize no # 或者注释daemonize yes
# daemonize yes 和 docker run 中-d 参数冲突

4 启动redis服务运行容器
docker run --name redis  -v /usr/local/redis/data:/data  -v /usr/local/redis/redis.conf:/usr/local/etc/redis/redis.conf -p 6379:6379 -d redis:6.0.10  redis-server /usr/local/etc/redis/redis.conf 

解释： 
# 将数据目录挂在到本地保证数据安全
# 宿主机绝对路径:容器内路径
-v /usr/local/redis/data:/data  
# 将配置文件挂在到本地修改方便
-v /root/redis/redis.conf:/usr/local/etc/redis/redis.conf   
 
5  直接进去redis客户端。
docker exec -it redis redis-cli

```

