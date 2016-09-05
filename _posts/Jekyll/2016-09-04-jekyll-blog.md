---
layout: post
title:  "使用Jekyll3和Coding Pages搭建静态博客"
date:   2016-09-04 1:28:54
categories: Jekyll
tags: Jekyll Ruby gem bundle
---

* content
{:toc}

三个月前兴冲冲的在万网申请了jisonami.org的域名，打算用jekyll将自己的博客放在Github Pages上，不过因为没有在本地将jekyll跑起来，然后一直拖到现在。

这两天花时间认真了解了下jekyll的用法，解决了之前遇到的一些问题，成功用jekyll搭了个静态博客并托管在Coding Pages服务上。

为何不用GitHub Page? Coding Pages毕竟是国内的服务，速度上有点优势。当然，GitHub Pages服务也是要同时使用的，不过我的独立域名jisonami.org绑定在Coding Pages上就是了。

记录一下使用jekyll时遇到的问题。



### jekyll用法

首先打开[jekyll中文官网](http://jekyll.bootcss.com/)

会看到里面介绍jekyll的安装和运行的用法，也就是以下几个命令：

```shell
 $ gem install jekyll
~ $ jekyll new my-awesome-site
~ $ cd my-awesome-site
~/my-awesome-site $ jekyll serve
# => Now browse to http://localhost:4000
```

**jekyll是基于ruby的，所以我们需要ruby**

### 安装ruby

#### centos7安装ruby

```
yum install ruby ruby-devel
```

#### windows安装ruby

下载[rubyinstaller](http://rubyinstaller.org/)

安装时需要将第二个选项勾上，即下图所示

![rubbyinstaller](/images/Jekyll/jekyll-blog/rubyinstaller.png)

### 更改ruby镜像源

为什么要更爱ruby镜像源，因为官方的[https://rubygems.org/](https://rubygems.org/)国内基本访问不了，也就是gem命令装不了任何包

淘宝的镜像[https://ruby.taobao.org](https://ruby.taobao.org)已经不维护了，现在维护的ruby镜像是[http://gems.ruby-china.org/](http://gems.ruby-china.org/)

删除默认的官方源

```
gem sources -r https://rubygems.org/
```

添加ruby-china源

```
gem sources -a http://gems.ruby-china.org/
```

查看当前源

```
gem sources -l
```

注意看到添加ruby-china远时用的是http协议，为什么不用https协议呢？因为在windows下会以下错误

```
ERROR:  While executing gem ... (Gem::RemoteFetcher::FetchError)
    SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://gems.ruby-china.org/specs.4.8.gz)
```


### 安装jekyll

```
gem install jekyll
```

如果是centos7安装jekyll的话，可能会报一下错误

```
Building native extensions.  This could take a while...
ERROR:  Error installing jekyll:
        ERROR: Failed to build gem native extension.

    /usr/bin/ruby extconf.rb
checking for ffi.h... *** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.
```
这是由于缺少gcc、g++依赖造成的，安装一下就好

```
yum install gcc g++
```

然后跟着jekyll官方的教程运行jekyll serve的时候会报一下错误

```
/usr/share/rubygems/rubygems/core_ext/kernel_require.rb:55:in `require': cannot load such file -- bundler (LoadError)
        from /usr/share/rubygems/rubygems/core_ext/kernel_require.rb:55:in `require'
        from /usr/local/share/gems/gems/jekyll-3.2.1/lib/jekyll/plugin_manager.rb:34:in `require_from_bundler'
        from /usr/local/share/gems/gems/jekyll-3.2.1/exe/jekyll:9:in `<top (required)>'
        from /usr/local/bin/jekyll:23:in `load'
        from /usr/local/bin/jekyll:23:in `<main>'
```

这是因为jekyll new新建的jekyll项目里面有个Gemfile文件，这个文件是ruby中bundle管理gem依赖包的方式，所以我们还需要安装bundle

### 安装bundle

```
gem install bundle
```

同样配置bundle的镜像源为ruby-china

```
bundle config mirror.https://rubygems.org http://gems.ruby-china.org/
```

### 安装Gemfile依赖

```
cd jekyllprojectdir
bundle install
```

这个时候运行jekyll serve就可以正常使用localhost:4000访问官方教程新建的jekyll静态网站了。

### 使用jekyll主题

在这里[http://jekyllthemes.org/](http://jekyllthemes.org/)可以找到很多jekyll的主题，找到喜欢的主题将源码下载下来，使用前面介绍的方式可以将jekyll项目运行起来就可以看到效果了

需要注意的是如果下载的主题源码中有Gemfile文件，需要bundle install一下

知乎有个帖子是关于jekyll主题讨论的，我博客这个主题也是在这个帖子里面找到的[有哪些简洁明快的 Jekyll 模板？](http://www.zhihu.com/question/20223939)

有些jekyll主题是有别的依赖的，比如我博客这个主题就会报这个错

```
D:\>cd D:\SDK\Ruby\gaohaoyang.github.io-master

D:\SDK\Ruby\gaohaoyang.github.io-master>jekyll serve
Configuration file: D:/SDK/Ruby/gaohaoyang.github.io-master/_config.yml
  Dependency Error: Yikes! It looks like you don't have jekyll-paginate or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- jekyll-paginate' If you run into trouble, you can find helpful resources at http://jekyllrb.com/help/!
jekyll 3.2.1 | Error:  jekyll-paginate
```

解决办法：

```
gem install jekyll-paginate
```

### 使用Coding Pages服务

这个比较简单，与Github Pages服务类似，先push代码

不过Github Pages服务只能是使用username.github.io这一个项目作为静态网站运行

所以我在Github Pages服务托管的博客就是[https://jisonami.github.io/](https://jisonami.github.io/)

而[Coding Pages服务](https://coding.net/help/doc/pages/index.html)提供了图形化的操作界面，并且每个项目的每个分支都能使用该服务

而在Coding Pages服务托管的博客使用了独立域名[jisonami.org](http://jisonami.org)