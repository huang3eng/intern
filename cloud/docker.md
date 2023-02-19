# docker

## 基本概念

- dockfile -> 源代码、image -> 程序、container -> 进程、docker -> 编译器
- repository这里是镜像仓库，是Docker用来集中存放镜像文件的地方，每个镜像利用tag进行区分
- registry注册服务器是存放仓库的地方，一般会有多个仓库
- docker client和docker demon可以运行在同一台机器上

## 八股文

- docker底层两种技术cgroup和namespace
  - linux操作系统通过cgroup可以设置cpu、内存和IO资源的配额
  - namespace管理这host中全局唯一的资源，可以让每个容器都觉得只有他自己在使用他。
- 容器间的通信：IP通信、Docker DNS Server和joined容器
- volume container和data-packed volume container

## [常见命令](https://docs.docker.com/engine/reference/commandline/cli/)

- `docker exec -it CONTAINER_ID bash` 进入容器终端并且的保留为容器终端的输入形式(-it和bash的结合作用)
- `docker build`
  - `--no-cache`：标志将简单地停止Docker引擎使用缓存层，它将重新下载和构建所有内容
  - `--tag`：镜像的名字及标签，通常name:tag
  - `--build-arg=[]` :设置镜像创建时的变量
  - `-f` :指定要使用的Dockerfile路径
- `docker run`
  - `--entrypoint`：运行时默认执行的命令
  - `-e`：指定环境变量，容器内部再去处理这个环境变量
  - `--rm`：在容器退出时就能够自动清理容器内部的文件系统、清理容器的匿名data volumes
  - `--gpus all`：容器使用全部gpu
  - `-it`：是两个参数：-i和-t。前者表示打开并保持stdout，后者表示分配一个终端（pseudo-tty）
  - -`d`：后台运行 不会对当前终端产生任何输出，所有的stdout都输出到log
  - `-v`：参数，冒号前为宿主机目录，必须为绝对路径，冒号后为镜像内挂载的路径
  - `--pid=host` ：这个选项的功能是让容器能够看到容器外面的主机的世界，也就是能通过ps aux 来查看主机上的进程
  - `--net=host`：这个是禁用了网络隔离，让我们容器共享主机网络，容器的IP地址和主机是一个IP地址
  - `--ipc=host`：这个是允许我们设置一些kernel的参数docker run –sysctl net.ipv4.ip_forward=1 someimage
  - `-p host-port:container-port`：容器里的端口container-port映射成主机的端口host-port
  
- `docker commit [container id] [new name:tag]`
  - 保存修改后的container为image

## dockfile

```Dockerfile
ENV TZ=Asia/Shanghai #设置镜像时区
ENV DEBIAN_FRONTEND noninteractive #ubuntu是基于apt工具包进行软件安装的，而apt工具包是基于DebianPackageManagement的。而DEBIAN_FRONTEND就是DebianPackageManagement的基本选项

# dockfile start
# 说明该镜像以哪个镜像为基础
FROM centos:latest

# 构建者的基本信息
MAINTAINER hlm

# 在build这个镜像时执行的操作
RUN yum update
RUN yum install -y git

# 拷贝本地文件到镜像中
COPY ./* /usr/share/gitdir/
# dockfile end

ADD 宿主机文件的全路径 docker容器下的文件夹路径

```

ARG 和 ENV 的区别

- ARG 定义的变量只会存在于镜像构建过程，启动容器后并不保留这些变量
- ENV 定义的变量在启动容器后仍然保留
