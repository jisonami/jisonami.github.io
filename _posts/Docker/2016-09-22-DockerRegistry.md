---
layout: post
title:  "跟我学Docker（三）--搭建Docker私有仓库服务"
date:   2016-09-22 23:14:54
categories: Docker
tags: Docker docker-registry
---

* content
{:toc}

Docker官方开源了搭建Docker Registry的项目。

有两个版本，一个是python实现的[Docker Registry](https://github.com/docker/docker-registry)
，适用于docker1.6及之前的版本，另一个是go实现的[Docker Registry V2](https://github.com/docker/distribution)，适用于Docker1.6及更新的版本。

如果不考虑Docker1.6，那么建议搭建Docker Registry V2。如果需要考虑兼容Docker1.6之前的版本，可以考虑同时搭建Docker Registry和Docker Registry V2，然后使用nginx做反向代理。

本文只介绍Docker Registry V2私有仓库服务的搭建以及docker-engine端如何使用私有仓库服务加速docker image的pull速度。




在此假设你已经安装好docker-engine，如果没有，请参考我的博文[跟我学Docker（一）--Docker如何安装](/2016/09/15/DockerInstall/)

请注意，以下内容以CentOS7为例，其它的Linux发行版可能需要找到对应的配置文件。

### 搭建Docker私有仓库服务

#### 为什么需要私有仓库

第一个原因是Docker Hub的pull速度实在是太慢了，第二个原因是企业内部构建的docker镜像不可能直接放到Docker Hub上。

没有私有仓库服务之前，pull请求是这样的

<img src='http://g.gravizo.com/g?
@startuml;
"docker engine" -> "Docker Hub": pull request;
"Docker Hub" --> "docker engine": return a image;
@enduml
'/>

使用私有仓库服务之后，pull请求是这样的

<img src='http://g.gravizo.com/g?
@startuml;
"docker engine" -> "Registry V2": pull request;
"Registry V2" --> "docker engine": return a image;
;
"docker engine" -> "Docker Hub": fail to pull request from Registry V2, pull request;
"Docker Hub" --> "docker engine": return a image;
@enduml
'/>

#### 搭建私有仓库

**一条命令部署Docker Registry V2**

```
docker run -d -p 5000:5000 -v /var/lib/registry:/var/lib/registry --restart=always --name registry-v2-test registry:2
```

现在一个Docker私有的仓库服务就搭建好了。

测试registry V2容器是否运行正常，浏览器访问localhost:5000看到“{}”这样的结果就正常了。

简单解释一下，上面的命令将registry容器的5000端口映射到本机的5000端口，并且将本机的/var/lib/registry目录挂载到registry容器的/var/lib/registry目录，该目录作为私有仓库存储docker镜像层的目录。

参考[Docker Registry V2官方部署文档](https://docs.docker.com/registry/deploying/)

**那么如何使用这个Docker私有仓库服务呢**

我们可以进行push和pull操作，当然，操作的Docker镜像的tag标记要求是以registry服务所在机器的地址开头的。可以是域名或者IP地址。

例如，在本机上运行的registry服务，那么就可以使用localhost:5000

如果我们有一个nginx:1.10.1的docker镜像在本地，我们需要对其重新打一个tag标记为localhost:5000/library/nginx:1.10.1，这个library表示是Docker Hub官方提供的镜像，

```
docker tag nginx:1.10.1 localhost:5000/library/nginx:1.10.1
```

然后才可以push到我们的registry私有仓库上了

```
docker push localhost:5000/library/nginx:1.10.1
```

现在我们的私有仓库上就有了一个localhost:5000/library/nginx:1.10.1这样的镜像了。对这个镜像镜像pull操作

```
docker pull localhost:5000/library/nginx:1.10.1
```

其实Docker Hub上的tag表示是docker.io/library/nginx:1.10.1, 但是官方镜像pull时可以省略docker.io/library/这一部分。

当然也可以使用IP地址来替代localhost，比如127.0.0.1，

如果是别的机器需要对该私有仓库进行push或pull操作，那么使用该私有仓库的IP地址就行了，比如192.168.0.169。

#### 私有仓库非安全问题

**在上面的操作过程中，你有可能会遇到无法push镜像到私有仓库的问题。这是因为我们启动的registry服务不是安全可信赖的**

我们需要在/etc/sysconfig/docker文件上配置一个参数INSECURE_REGISTRY

使用vi编辑/etc/sysconfig/docker文件

```
vi /etc/sysconfig/docker
```

配置该参数

```
INSECURE_REGISTRY='--insecure-registry 192.168.0.169:5000'
```

重启docker服务

```
systemctl daemon-reload
systemctl restart docker
```

#### 添加私有仓库

现在问题来了，我们每次在docker私有服务上pull镜像时都需要使用localhost:5000/library/nginx:1.10.1这样长的tag，可不可以把私有服务的地址去掉呢？答案是可以的。

在docker-engine的配置文件/etc/sysconfig/docker中配置参数ADD_REGISTRY

```
ADD_REGISTRY='--add-registry 192.168.0.169:5000'
```

同样重启docker服务

```
systemctl daemon-reload
systemctl restart docker
```

先把原来pull下载的localhost:5000/library/nginx:1.10.1删掉

```
docker rmi localhost:5000/library/nginx:1.10.1
```

现在我们就可以使用library/nginx:1.10.1来替代localhost:5000/library/nginx:1.10.1了

```
docker pull library/nginx:1.10.1
```

#### 添加Docker Hub的镜像仓库

为什么不能像使用Docker Hub一样把library也去掉呢，前面说了library前缀表示docker官方维护的镜像，其实只在Docker Hub上才是这样，在私有仓库上这个library就不能省略了。当然我们自己的docker镜像也不会命名为library开头的tag。

当我们的registry私有仓库服务作为Docker Hub的镜像仓库mirror registry时，这个library也就可以省略掉了。

配置docker mirror registry步骤如下：

```
cp /lib/systemd/system/docker.service /etc/systemd/system/docker.service
```

用vi编辑/etc/systemd/system/docker.service

```
vi /etc/systemd/system/docker.service
```

找到下面这一行

```
ExecStart=/usr/bin/docker-current daemon
```

在后面加上--registry-mirror=http://192.168.0.169:5000, 即

```
ExecStart=/usr/bin/docker-current daemon --registry-mirror=http://192.168.0.169:5000
```

重启docker服务

```
systemctl daemon-reload
systemctl restart docker
```

现在我们就可以直接这样从registry私服仓库上pull镜像了，当私有仓库上找不到时，依旧会在Docker Hub上去找

```
docker pull nginx:1.10.1
```

现在问题又来了，之前的例子中的nginx:1.10.1镜像是我们手动tag命名后手动push到registry私服仓库上的。

Docker Hub上这么多镜像我们不可能手动pull下载，tag之后再push到私有仓库上吧？

据说，阿里云和daocloud的加速器服务就是这么实现，只不过把手动的操作改用程序实现。

事实上Registry是对于所有的命令行的操作是提供了等价的restful接口的，通过这些接口就可以实现了。


### 使用nginx作为私有仓库服务的反向代理

为了不使Registry V2私有仓库服务的端口直接暴露出来，我们可以使用nginx最为私有仓库服务的反向代理服务器，接收到用户的pull请求后将其转发到Registry V2私有仓库服务，nginx将请求结果返回给用户。


同样，我们直接使用docker运行一个nginx容器就可以了。

```
docker run -d -p 80:80 -p 443:443 --name nginx-test nginx:1.10.1
```

现在，我们在浏览器访问localhost或127.0.0.1或本机ip地址192.168.0.169就可以看到nginx提供的index页面了。

但是呢，这样以默认配置运行的nginx容器并不能满足我们的要求，我们需要将nginx容器的配置目录从容器中复制出来，然后修改配置文件后，将该目录挂载到该配置目录上重新运行一个nginx容器

**将nginx配置目录复制到当前目录**

```
docker cp nginx-test:/etc/nginx .
```

修改nginx配置文件

```
vi nginx/conf.d/default.conf
```

**找到这一段配置**

```
location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```

修改成

```
location / {
        proxy_pass http://registry-v2:5000;
        #root   /usr/share/nginx/html;
        #index  index.html index.htm;
    }
```

现在把正在运行的nginx-test容器停止并删除

```
docker stop nginx-test && docker rm nginx-test
```

重新运行一个新的nginx容器

```
docker run -d --link registry-v2-test:registry-v2 -v `pwd`/nginx:/etc/nginx -p 80:80 -p 443:443 --name nginx-test nginx:1.10.1
```

参数--link registry-v2-test:registry-v2将registry-v2-test私有仓库容器映射为registry-v2，这样nginx-test容器就可以通过registry-v2直接访问registry-v2-test容器了

--link参数实际上是将registry-v2写到nginx-test容器的/etc/hosts文件上了。我们可以通过以下命令看到

```
[root@localhost ~]# docker exec -it nginx-test cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	registry-v2 b61e5b5c16a6
172.17.0.3	e4daaf7209c3
```

这个172.17.0.2和172.17.0.3分别是正在运行的容器registry-v2-test和nginx-test的ip，我们可以通过下面的命令查看一下容器的ip

```
[root@localhost ~]# docker inspect -f '{{.NetworkSettings.IPAddress}}' registry-v2-teset nginx
172.17.0.2
172.17.0.3
```

参数-v `pwd`/nginx:/etc/nginx将当前目录的nginx目录映射到nginx-test容器的/etc/nginx目录上

现在在浏览器直接访问localhost/v2就可以看到之前使用localhost:5000/v2看到一样的结果了。

那么，docker-engine端的相关配置INSECURE_REGISTRY、ADD_REGISTRY、--registry-mirror就可以去掉端口了。

即http://192.168.0.169:5000就可以变成http://192.168.0.169了。

记得重启docker服务

```
systemctl daemon-reload
systemctl restart docker
```

**如果我们有个域名叫“jisonami.org”，我们在运行nignx容器是加个参数--hostname**

```
docker run -d --hostname jisonami.org --link registry-v2-test:registry-v2 -v `pwd`/nginx:/etc/nginx -p 80:80 -p 443:443 --name nginx-test nginx:1.10.1
```

其实这个--hostname参数同样是修改了/etc/hosts文件

```
[root@localhost ~]# docker exec -it nginx cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	registry-v2 b61e5b5c16a6
172.17.0.3	jisonami.org
```

我们在本机CentOS7上同样需要配置/etc/hosts文件

```
vi /etc/hosts
```

增加一行

```
172.17.0.3	jisonami.org
```

如果是别的机器，则需要使用IP地址192.168.0.169

```
192.168.0.169 jisonami.org
```

现在在浏览器直接访问jisonami/v2就可以看到之前使用localhost:5000/v2看到一样的结果了。

那么，docker-engine端的相关配置INSECURE_REGISTRY、ADD_REGISTRY、--registry-mirror就可以修改成jisonami.org

即http://192.168.0.169:5000就可以变成http://jisonami.org了。

记得重启docker服务

```
systemctl daemon-reload
systemctl restart docker
```

同样的，我们如果需要使用二级域名或者三级域名如法炮制即可。

当然，我们现在是通过修改/etc/hosts文件的形式模拟域名的使用，若是公网可以直接访问该域名或者局域网内部搭建了域名服务器则不需要修改/etc/hosts文件了。
