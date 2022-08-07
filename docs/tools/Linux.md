# 文件夹组成

<img src="https://cdn.jsdelivr.net/gh/YiENx1205/cloudimgs/notes/202204240904183.png" alt="iShot2022-04-24_09.02.28" style="zoom: 40%;" />

1. **bin**

	存放的是普通用户使用的命令。如 yum zip mkdir ... ，与之对应的是sbin，存放的是管理员使用的命令。

2. **boot**

	存放的是系统启动相关的核心文件，例如grub(引导安装的程序)

3. **dev**

	存放的是一些计算机设备文件

4. **etc**

	存放的是一些配置文件

5. **home**

	用来存放非root用户的家目录的，也就是登录普通用户后主机名后是“~”时所在的目录

6. **root**

	root用户家目录的

7. **run**

	存放各种运行的数据，是运行中进程产生的文件

8. **sbin**

	存放的是管理员使用的命令

9. **tmp**

	存放临时文件的，进程产生的文件

10. **usr**

	存放系统文件，相当于windows 的C:/windows 文件。

11. **var**

	存放的是一写变化的文件，如日志、数据库等

12. **opt**

	这个是给第三方协力软件放置的目录

# 常用命令

显示xxx相关的进程

```shell
ps -ef|grep xxx
```

基于rpm安装

```shell
# 列出所有可以安装的软件包
yum list
# 安装软件
yum install -y 软件名
# 卸载软件
yum remove 软件名
# 查找软件包
yum search all 软件名
```

服务相关

```shell
# 服务器管理命令
systemctl status 服务名
# 启动服务
systemctl start 服务名
# 重启服务
systemctl restart 服务名
# 停止服务
systemctl stop 服务名
# 禁止服务随linux启动。
systemctl disable 服务名
# 设置服务随linux启动。
systemctl enable 服务名
```

查看cpu使用效率[top](https://blog.csdn.net/u012102536/article/details/122221926)

```shell
top
```

杀死程序

```shell
kill
```

> **kill -15** **和** **kill -9** **的区别**
>
> kill -9 PID 是操作系统从内核级别**强制杀死一个进程****.** killed
>
> kill -15 PID 可以理解为操作系统**发送一个通知告诉应用主动关闭**. terminated

