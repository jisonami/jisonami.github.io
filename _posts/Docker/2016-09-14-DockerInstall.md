---
layout: post
title:  "跟我学Docker（一）--Docker如何安装"
date:   2016-09-14 23:14:54
categories: Docker
tags: Docker docker-engine
---

* content
{:toc}

网上一搜，关于Docker安装的文章都太老了，只能安装docker1.9及之前的版本，而现在docker最新版本已经到了docker1.12了

本文介绍docker当前新版本的安装方式，包括centos7、ubuntu16.04、deepin15以及windows7不同操作系统的安装方式，请原谅我没有Mac Book

请确保您拥有root权限或者管理员权限，下面开始吧




原来我们安装docker在centos下是这么安装的

```
yum install docker
```

这种方式只能安装docker1.9之前的版本

原来我们安装docker在ubuntu下是这么安装的

```
apt-get install docker
```

这种方式之只能安装docker1.5及之前的版本

或者这样

```
apt-get install docker.io
```

这种方式只能安装docker1.9及之前的版本

随着docker架构的发展，docker的模块结构也发生了变化，目前docker的安装是通过安装docker-engine包来实现的

下面介绍docker-engine的安装方式


### CentOS下安装docker-engine

#### 使用yum安装docker-engine

**确保你的linux内核和yum包是最新的**

```
yum update
```

**配置docker的yum源**

终端输入如下命令

```
tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```

**刷新yum源**

```
yum clean all
yum makecache
```

**安装docker-engine**

```
yum install docker-engine
```

**启动docker后台进程**

```
systemctl start docker
```

**将docker后台进程加入开机自启**

```
systemctl enable docker
```

**查看已安装docker版本**

```
docker -v
```

#### 使用docker官方脚本安装

**确保你的linux内核和yum包是最新的**

```
yum update
```

**运行docker安装脚本**

```
curl -fsSL https://get.docker.com/ | sh
```

**启动docker后台进程**

```
systemctl start docker
```

**将docker后台进程加入开机自启**

```
systemctl enable docker
```

**查看已安装docker版本**

```
docker -v
```

参考官方安装文档[centos安装docker-engine](http://docs-stage.docker.com/engine/installation/linux/centos/)


### ubuntu16.04安装docker-engine

**配置软件源**

```
deb https://apt.dockerproject.org/repo ubuntu-xenial main
```

**更新软件源索引**

```
apt-get update
```

**安装linux内核补丁**

```
apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
```

**安装docker-engine**

```
apt-get install docker-engine
```

**启动docker后台进程**

```
systemctl start docker
```

**将docker后台进程加入开机自启**

```
systemctl enable docker
```

**查看已安装docker版本**

```
docker -v
```

参考官方安装文档[ubuntu安装docker-engine](http://docs-stage.docker.com/engine/installation/linux/ubuntulinux/)

### deepin15安装docker-engine

**配置软件源**

```
deb https://apt.dockerproject.org/repo debian-stretch main
```

**更新软件源索引**

```
apt-get update
```

**安装docker-engine**

```
apt-get install docker-engine
```

**启动docker后台进程**

```
systemctl start docker
```

**将docker后台进程加入开机自启**

```
systemctl enable docker
```

**查看已安装docker版本**

```
docker -v
```

deepin15的内核是基于debain sid再打包的，docker-engine安装可以参考[debian安装docker-engine](http://docs-stage.docker.com/engine/installation/linux/debian/#/debian-wheezy-stable-7-x-64-bit)

### windows7安装Docker Toolbox

在windows上，使用docker是通过在virtual中使用boot2docker.iso创建虚拟机的方式来使用docker的

原来我们都是手动管理boot2docker的方式，这种方式在网上已经很多文章描述了

Docker Toolbox集成了很多docker的操作工具，包括git、virtualbox、docker-quickstart-terminal                            等工具，Docker Quickstart Terminal是Docker Toolbox很方便用来管理docker的工具

现在介绍Docker Toolbox的方式在windows上来使用docker

[下载Docker Toolbox](https://www.docker.com/products/docker-toolbox)

安装很简单，一直下一步下一步就行了

当然如果你和我一样在天朝，可能就会遇到下面的情况

安装完Docker Toolbox后

启动Docker Quickstart Terminal，发现VirtualBox没有安装，找到Docker Toolbox安装目录下的installers/virtualbox目录，安装virtualbox.msi

重新启动Docker Quickstart Terminal，自动更新boot2docker最新版

发现自动更新失败，到[https://github.com/boot2docker/boot2docker/releases](https://github.com/boot2docker/boot2docker/releases)下载最新的boot2docker.iso，依旧启动失败

错误信息如下

```
Running pre-create checks...
(default) Default Boot2Docker ISO is out-of-date, downloading the latest release
...
(default) Latest release for github.com/boot2docker/boot2docker is v1.12.1
(default) Downloading C:\Users\jishengxu\.docker\machine\cache\boot2docker.iso f
rom https://github.com/boot2docker/boot2docker/releases/download/v1.12.1/boot2do
cker.iso...
```

解决办法：把网络断掉，重新启动Docker Quickstart Terminal，一切正常了，断网第一次启动Docker Quickstart Terminal后就可以把网络打开了

默认情况下，Docker Toolbox创建的default虚拟机是在C盘下的C:\Users\username\\.docker\machine\machines\default                                                     目录，里面有个disk.vmdk           的虚拟硬盘文件，我们pull命令下来的所有docker镜像都是存放在这个disk.vmdk里面。

所以我们可以像个办法将default的disk.vmdk虚拟硬盘从C盘移到别的盘，然后在virtual中将虚拟硬盘指向新的disk.vmdk，这是会遇到以下错误

错误：打开虚拟硬盘失败，UUID already exists

virtualbox在命令行修改虚拟硬盘uuid命令

```
vboxmanage.exe internalcommands sethduuid D:\yourDir\disk.vmdk
```

然后重新就指向disk.vmdk这个虚拟硬盘文件就好了

参考官方安装文档[windows安装docker-engine](http://docs-stage.docker.com/engine/installation/windows/)

### 使用阿里云的docker mirror加速器

默认情况下，我们在docker hub上pull镜像时速度是很慢的，幸好现在有阿里云提供的docker mirror加速器来提高pull镜像的速度，配好加速器后pull速度可以达到1M以上，之前有个daocloud docker mirror在docker-engine中不太好用了。

在[dev.aliyun.com](dev.aliyun.com)注册一个账号，然后打开管理中心，找到加速器页面，里面有详细ubuntu、centos、mac os x和windows下的加速器使用方式，就不罗嗦介绍了