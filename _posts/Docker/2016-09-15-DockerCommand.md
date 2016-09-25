---
layout: post
title:  "跟我学Docker（二）--Docker的基本用法"
date:   2016-09-15 23:14:54
categories: Docker
tags: Docker docker镜像 docker容器
---

* content
{:toc}

vmware下的RHEL/CentOS的网络配置请参考：[http://jisonami.iteye.com/blog/2306735](http://jisonami.iteye.com/blog/2306735)

RHEL/CentOS在7系列才完全的原生支持docker，以前全部例子均使用CentOS7演示，并且所有操作在root权限下进行。

本文介绍docker最常用的基本用法，docker常用命令以及docker镜像与容器的基本操作。




### docker常用命令介绍

### docker镜像与容器操作实战

docker的三个关键词：仓库、镜像、容器

这三者的关系是这样的，仓库存放着很多镜像，每个镜像可以运行很多个容器

官方仓库是http://index.docker.io

#### 操作docker镜像的相关命令及用法演示

**从仓库查找镜像**

命令：docker search 镜像名

以查找redis为例

```
docker search redis
```

**从仓库下载镜像到本地**

命令：docker pull 镜像名

带标签名，以下载redis镜像为例，

```
docker pull redis:2.8.21
```

若不带标签名，则默认下载最新（latest）版本

```
docker pull redis
```

**查看镜像列表**

```
docker images
```

**删除镜像**

命令：docker rmi 镜像id

已删除redis镜像为例

先查看镜像列表，找到要删除的镜像id

```
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker.io/redis     2.8.21              1a721decd792        10 months ago       109 MB
```

删除IMAGE ID为1a721decd792的redis镜像

```
docker rmi 1a721decd792
```

**删除所有镜像**

```
docker rmi $(docker images -q)
```

**镜像的导出导入**

导出命令：docker save -o tar包名字 镜像仓库:镜像标签

示例：

```
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker.io/redis     2.8.21              1a721decd792        10 months ago       109 MB
[root@localhost ~]# docker save -o redis2.8.21.tar docker.io/redis:2.8.21
[root@localhost ~]# ls
anaconda-ks.cfg  redis2.8.21.tar
```

导入命令：docker load < tar包名字

示例：先删除原有的镜像再导入，删除镜像前需要先删除基于该镜像的而所有容器

```
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
985b07080de0        1a721decd792        "/entrypoint.sh redis"   52 minutes ago      Exited (0) 26 minutes ago                       tiny_mccarthy
[root@localhost ~]# docker rm 985b07080de0
985b07080de0
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker.io/redis     2.8.21              1a721decd792        10 months ago       109 MB
[root@localhost ~]# docker rmi 1a721decd792
Untagged: docker.io/redis:2.8.21
Deleted: 1a721decd792ad1d3c4ebc315d34b1f8d4dfb97e6d8013efe6c523e637361bd6
Deleted: 1e0f123a26af5e26e27b7bcef888957657e21992f94beeb7f15b311731a94522
Deleted: 917499e60e0beb9aef579ab3bda18852c3dae3068608464a442fd1004929f350
Deleted: bdd393eaafef5acb1fa55708831d05d4203619bb46a84cf687fe9d85ce01d4e7
Deleted: 3d78ebcf73261a592159a96275f31eb08165ed128d7a775d97725d8d11e7e1ef
Deleted: 2da1436f270c273773122e93f8668672b8b7923af8d68374de4cfa5feeaeccd3
Deleted: fb03d161db1463c2208715b1df9af461a00cb2b74b64478ec131f727c5204629
Deleted: 87b88b774507ab4e2ee76c90b00f6fd9a0c8e7457c265e8256fcb6ac4fef8e28
Deleted: 6846105605cc29e8e7a65244961d5f82343ec2fb2ac824372ec23c198318a3b4
Deleted: d60d207d76b30a03d1da1100a2ed24f8a798104c73be5058ddb30abc6145be48
Deleted: b2508f29bec6010c47e4ca1ebd899252baa8e0645421017728897322368bbb34
Deleted: 7bf6d72c51dd7602557d3f7c1474f659d56819444a49a7f924df8e014cd756c3
Deleted: 2658b5be63c6886e82374815fd97e0593976fda3c749f531c88d2ec968d40709
Deleted: 047a294381b2ec81cbfb1d5710b65d641548b99d5e35dab761179ccddc36bacd
Deleted: 9bce9474876b400ca739a85e246c2677ebacc8440558fab04c764a76c7a52d78
Deleted: bbe78c1a5a535fac669e3225d5c3bb4396b6b2f9decb560ffb6351396da8c345
Deleted: 9d3ceacde91b4c7d6c1275032adb558d668afd5489c007f3512b39793ddf992d
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
[root@localhost ~]# docker load < redis2.8.21.tar
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
docker.io/redis     2.8.21              1a721decd792        10 months ago       109 MB
```

注意：

使用镜像id代替“镜像仓库:镜像标签”导出镜像也是可以的，但是会丢失元数据（即仓库信息和标签名等）

重新导入后看到的信息如下

```
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
<none>              <none>              1a721decd792        10 months ago       109 MB
```

可以看到导入的镜像居然没有元数据（即仓库信息和标签名等），这是因为导出的时候使用的是镜像id导致的

解决方式是导出镜像时使用“镜像仓库:镜像标签”而不是镜像id

#### 操作docker容器的相关命令及用法演示

**查看运行中的容器**

```
docker ps
```

查看所有容器包括已关闭的容器

```
docker ps -a
```

**从镜像中启动一个容器**

命令：docker run 选项 镜像名 <命令>

镜像后的命令可省略

以启动redis容器为例

```
docker run -d -p 6379:6379 1a721decd792
```

启动redis容器后，我们可以使用redis-cli命令行客户端连接redis服务器

```
[root@localhost ~]# redis-cli
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379>
```

参数解释：

-d 表示启动一个后台运行的容器

-p 表示指定容器端口号，指定的格式可以是

ip:主机端口:容器端口| ip::容器端口 | 主机端口:容器端口 | 容器端口

若没有指定主机端口，则随机生成

以启动bash并获得控制台标准输入为例

```
docker run -t -i 1a721decd792 /bin/bash
```

参数解释：

-t	让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上

-i	让容器的标准输入保持打开

**关闭运行中的容器**

命令：docker stop 容器id

以关闭刚刚启动的redis容器为例

首先通过docker ps命令查找运行中的容器信息，根据查找的容器id关闭该容器

```
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
985b07080de0        1a721decd792        "/entrypoint.sh redis"   6 seconds ago       Up 4 seconds        0.0.0.0:6379->6379/tcp   tiny_mccarthy
[root@localhost ~]# docker stop 985b07080de0
985b07080de0
```

**启动已关闭的容器**

命令：docker start 容器id

以启动刚刚关闭的redis容器为例

首先通过docker ps -a命令查找所有的容器信息，根据查找的容器id关闭该容器

```
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
985b07080de0        1a721decd792        "/entrypoint.sh redis"   2 minutes ago       Exited (0) 2 minutes ago                       tiny_mccarthy
[root@localhost ~]# docker start 985b07080de0
985b07080de0
```

**重启正在运行中的容器**

命令：docker restart 容器id

首先通过docker ps命令查找运行中的容器信息，根据查找的容器id关闭该容器

```
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
985b07080de0        1a721decd792        "/entrypoint.sh redis"   6 seconds ago       Up 4 seconds        0.0.0.0:6379->6379/tcp   tiny_mccarthy
[root@localhost ~]# docker restart 985b07080de0
985b07080de0
```

**进入运行中的后台docker容器**

1）attach命令

docker attach 容器id

（以redis为例，若使用docker attach命令则直接连到redis-server那个输入流，不管开几个docker attach都只连接到该输入流，并且同步刷新所有窗口）

这里以启动bash并获得控制台标准输入为例，以为redis容器不直观

```
[root@localhost ~]# docker run -d -t -i 1a721decd792 /bin/bash
168a188ae07eb437d2ef33f22a30624c667c3eea2be5300ef51641df440c1610	#启动容器是返回的完整的容器id，这里使用docker ps查询的短容器id
Usage of loopback devices is strongly discouraged for production use. Either use `--storage-opt dm.thinpooldev` or use `--storage-opt dm.no_warn_on_loop_devices=true` to suppress this warning.
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
168a188ae07e        1a721decd792        "/entrypoint.sh /bin/"   22 seconds ago      Up 21 seconds       6379/tcp            clever_saha
[root@localhost ~]# docker attach 168a188ae07e
root@168a188ae07e:/data# pwd
/data
```

2）nsenter命令

安装util-linux包，nsenter命令在这个包里面,centos7仓库里的util-linux正好是连接docker容器所需的2.23版本

```
yum install util-linux
```

docker获取pid

```
docker inspect --format "{{ .State.Pid}}" 容器id
```

使用nsenter命令连接

```
nsenter --target 3212 --mount --uts --ipc --net --pid
```

完整的例子：（以redis为例，若使用docker attach命令则直接连到redis-server那个输入流，不管开几个docker attach都只连接到该输入流，并且同步刷新所有窗口）

```
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
985b07080de0        1a721decd792        "/entrypoint.sh redis"   20 minutes ago      Up 6 seconds        0.0.0.0:6379->6379/tcp   tiny_mccarthy
[root@localhost ~]# docker start 985b07080de0
985b07080de0
[root@localhost ~]# docker inspect --format "{{ .State.Pid}}" 985b07080de0
8660
[root@localhost ~]# nsenter --target 8660 --mount --uts --ipc --net --pid
root@985b07080de0:/# redis-cli
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379>
```

**删除已关闭的容器**

命令：docker rm 容器id

这里以删除第一条记录为例

```
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS               NAMES
168a188ae07e        1a721decd792        "/entrypoint.sh /bin/"   4 minutes ago       Exited (130) 33 seconds ago                       clever_saha
985b07080de0        1a721decd792        "/entrypoint.sh redis"   16 minutes ago      Exited (0) 7 minutes ago                          tiny_mccarthy
[root@localhost ~]# docker rm 168a188ae07e
168a188ae07e
```

**删除所有已关闭的容器**

```
docker rm $(docker ps -a -q)
```

**容器导出和导入**

1）容器导出为tar包

命令：docker export 容器id > 导出tar包名字

示例：

```
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
985b07080de0        1a721decd792        "/entrypoint.sh redis"   27 minutes ago      Exited (0) 18 seconds ago                       tiny_mccarthy
[root@localhost ~]# docker export 985b07080de0 > redis.tar
[root@localhost ~]# ls
anaconda-ks.cfg  redis.tar
```

2）从tar包导入镜像

这里导入之后是镜像，而不是容器，并且需要重命名镜像元数据

命令：cat export导出的tar包 | docker import - 本地仓库名:标签名

示例：

```
[root@localhost ~]# cat redis.tar | docker import - testimport/redis:2.8.21
3d970cce18b58b71b5f83cb2778649a5b36c07de5cc824a758dcc2000668740b
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
testimport/redis    2.8.21              3d970cce18b5        6 seconds ago       107.3 MB
docker.io/redis     2.8.21              1a721decd792        10 months ago       109 MB
```

3）从导入的镜像启动容器

前面演示的时候启动redis容器是没有加命令的，导入的镜像启动容器时必须在末尾加上命令，否则会启动失败

即原先是这样的docker run -d -p 6379:6379 3d970cce18b5

现在必须这样docker run -d -p 6379:6379 3d970cce18b5 redis-server

示例：

```
[root@localhost ~]# docker run -d -p 6379:6379 3d970cce18b5 redis-server
7ad52d24fb8cad1d4bcb302355cbcd3e3d0ccb44ff6bdf5cb096271ecfc92419
Usage of loopback devices is strongly discouraged for production use. Either use `--storage-opt dm.thinpooldev` or use `--storage-opt dm.no_warn_on_loop_devices=true` to suppress this warning.
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
7ad52d24fb8c        3d970cce18b5        "redis-server"      3 seconds ago       Up 2 seconds        0.0.0.0:6379->6379/tcp   cranky_poincare
```

这里有一个坑

docker ps查看到的命令也许是错的，可能是因为显式的长度限制，比如

```
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS               NAMES
168a188ae07e        1a721decd792        "/entrypoint.sh /bin/"   4 minutes ago       Exited (130) 33 seconds ago                       clever_saha
985b07080de0        1a721decd792        "/entrypoint.sh redis"   16 minutes ago      Exited (0) 7 minutes ago                          tiny_mccarthy
```

启动时使用的是/bin/bash命令，这里显式的是"/entrypoint.sh /bin/"，按道理应该是"/entrypoint.sh /bin/bash"

启动redis时没带命令，这里显式的是"/entrypoint.sh redis"，按道理应该是"/entrypoint.sh redis-server"

请注意，save/load导出导入镜像和export/import导出容器快照导入镜像是不能混用的

redis2.8.21.tar 是save导出的镜像

redis.tar是export导出的容器快照

```
[root@localhost ~]# ls
anaconda-ks.cfg  redis2.8.21.tar  redis.tar
[root@localhost ~]# cat redis2.8.21.tar | docker import - docker.io/redis:2.8.21
Error response from daemon: EOF
[root@localhost ~]# docker load < redis.tar
Error response from daemon: open /var/lib/docker/tmp/docker-import-994843880/repo/bin/json: no such file or directory
```
