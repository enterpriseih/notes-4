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



# 使用

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
# -d：后台运行容器，并返回容器ID
# -i：以交互模式运行容器，通常与-t同时使用
# -t：为容器重新分配一个伪输入终端
# -p：指定端口映射，格式为："主机端口:容器端口"
# -P：随机端口映射，容器内部端口随机映射到主机的端口
# -v，--volume：指定卷的映射，格式为："主机卷:容器卷"
# --name="new name"：为容器指定一个名称
# --dns 8.8.8.8：指定容器使用的DNS服务器，默认和宿主一致
eg：
docker run -it nginx:latest /bin/bash
docker run –id –v /本地路径:/data/db –name 容器名 镜像

# 容器的启动关闭重启
docker start CONTAINER [CONTAINER...]
docker stop CONTAINER [CONTAINER...]
docker restart CONTAINER [CONTAINER...]

# 查看启动的容器
docker ps

# 进入容器
docker exec –it 容器名(或id) /bin/bash


```

