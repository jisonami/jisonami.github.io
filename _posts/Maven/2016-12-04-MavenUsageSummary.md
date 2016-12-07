---
layout: post
title:  "Maven详细用法"
date:   2016-12-04 23:14:54
categories: Maven
tags: Maven Java
---

* content
{:toc}

Maven是一个基于项目对象模型（POM，project object model）理念的综合软件项目管理工具，可以管理项目的自动化构建、测试、发布，以及文档、构建报告、项目质量信息的生成等一系列操作。




### Maven简介

Maven是一个基于项目对象模型（POM，project object model）理念的综合软件项目管理工具，可以管理项目的自动化构建、测试、发布，以及文档、构建报告、项目质量信息的生成等一系列操作。

#### Maven的目标

* 让项目构建更加简单
* 提供一个统一的构建系统
* 提供项目质量信息
* 提供最佳开发实践的指导
* 允许项目透明的迁移到新特性

#### Maven的特性

* 简单的配置就能得到一个最佳实践的项目或模块
* 不同maven项目一致性的用法使开发人员快速融入项目中
* 卓越的依赖关系管理包含自动更新依赖、依赖传递
* 非常容易同时处理多个项目
* 一个庞大并且持续增长的中央仓库，可以实时获取开源项目的最新版本的包依赖管理。
* 高度可扩展，很容易使用Java或脚本语言编写插件
* 基于模型的构建:Maven可以任意数量的项目构建到预定义的输出类型如一个jar、war,而不需要做任何脚本在大多数情况下。
* Maven可以生成一个网站或PDF文档包括文档、构建报告、项目质量信息等。
* 依赖关系管理:Maven鼓励使用中央存储库的jar和其他依赖项。Maven附带了一个机制,您的项目的客户可以使用下载所需的任何JAR从中央JAR库构建您的项目就像Perl语言的CPAN。这允许用户的Maven跨项目和鼓励重用jar项目之间的沟通,确保向后兼容性问题处理。
	
###	Maven安装

Maven查找settings.xml路径的顺序

$M2_HOME/conf/settings.xml --> $USER_HOME/.m2/settings.xml

#### Windows安装

首先去Apache的Maven官网下载最新版本的Maven：http://maven.apache.org/download.cgi 

![MavenDownload.png](/images/Maven/MavenUsageSummary/MavenDownload.png)

maven最新版本是3.3.9，我用的是3.3.3，懒得下载了，不要在意这些细节。 

将下载的zip压缩包解压到某个目录，这里将Maven解压到D:\SDK\Maven\apache-maven-3.3.3目录 

将解压后的Maven目录配置到环境变量中去，以Maven目录为D:\SDK\Maven\apache-maven-3.3.3为例，即如下图所示 

配置环境变量M2_HOME为：D:\SDK\Maven\apache-maven-3.3.3 

![M2_HOME.png](/images/Maven/MavenUsageSummary/M2_HOME.png)

配置环境变量Path为：%M2_HOME%\bin

![Maven_Path.png](/images/Maven/MavenUsageSummary/Maven_Path.png)

验证Maven安装，windows下打开cmd，执行mvn -version 

![mvn-version1.png](/images/Maven/MavenUsageSummary/mvn-version1.png)

发现提示没找到JAVA_HOME，那么配置一下环境变量JAVA_HOME 

![JAVA_HOME.png](/images/Maven/MavenUsageSummary/JAVA_HOME.png)

配置好JAVA_HOME变量后，重新打开cmd执行mvn –version 

![mvn-version2.png](/images/Maven/MavenUsageSummary/mvn-version2.png)

至此，maven安装成功了。 

#### CentOS安装

默认你已经在CentOS中把Java安装好

首先去Apache的Maven官网下载最新版本的Maven：http://maven.apache.org/download.cgi 

![MavenDownload.png](/images/Maven/MavenUsageSummary/CentOSInstall/MavenDownload.png)

将下载的zip压缩包解压到某个目录，这里将Maven解压到/usr/java/maven目录 

```
mkdir /usr/java
tar -xzvf apache-maven-3.3.9-bin.tar.gz -C /usr/java/maven
```

配置环境变量，使用vi打开/etc/profile文件

```
vi /etc/profile
```

在文件末尾添加以下两行

```
export M2_HOME=/usr/java/maven/apache-maven-3.3.9
export PATH=$PATH:$M2_HOME/bin
```

重新加载配置文件

```
source /etc/profile
```

验证Maven安装

```
mvn -v
```

![mvn-version.png](/images/Maven/MavenUsageSummary/CentOSInstall/mvn-version.png)


### 第一个Maven项目

以Windows 7为例，Win键 + R，出入cmd

![cmd.png](/images/Maven/MavenUsageSummary/cmd.png)

出入以下命令创建一个maven项目

```
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DarchetypeCatalog=local
```

现在我们就生成了一个helloworld级别的maven项目，有一个my-app的目录在当前操作系统的用户目录下，在C:/Users/YourName目录下

```
cd my-app
```

查看一下my-app的目录结构如下

```
    my-app
    |-- pom.xml
    `-- src
        |-- main
        |   `-- java
        |       `-- com
        |           `-- mycompany
        |               `-- app
        |                   `-- App.java
        `-- test
            `-- java
                `-- com
                    `-- mycompany
                        `-- app
                            `-- AppTest.java
```

/src/main/java是java的源码目录，/src/test/java是java的测试源码目录

打开App.java和AppTest.java就可以看到我们非常熟悉的Java代码和junit测试代码了，如果你没接触过maven，唯一陌生的就是pom.xml文件了

打开pom.xml文件看到如下信息

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>my-app</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

这个文件就是项目对象模型（POM，project object model），项目根标签是一个\<project\>标签，目前生成的这个pom.xml文件只包含本项目信息的描述和一个junit依赖关系的描述

这里面都用到了三个共同的标签元素，就是\<groupId/\>、\<artifactId/\>、\<version/\>，这三个元素就可以唯一确定一个项目或jar的坐标

\<groupId/\>用于描述一个组织或项目的唯一标识，一般使用公司或组织域名，或者域名再加上项目名（即这里的com.mycompany.app）

\<artifactId/\>用于描述一个项目或模块名，这里使用my-app，这需要确保在groupId下是唯一的就可以了

\<version/\>用于描述一个项目或模块的版本号

另外\<packaging/\>用于描述项目或模块的打包类型

\<name/\>和\<url/\>用于说明项目信息，并没有什么特殊作用

\<scope/\>用于描述依赖关系的作用范围，当前的test表示只用于测试代码范围

上面简单说明了pom.xml的作用，后面还会更加详细的说明

现在我们可以干什么？构建，这是maven的关键功能之一

```
mvn package
```

第一次执行这个命令后会出现以下信息开头

```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building my-app 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
Downloading: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-resources-plugin/2.6/maven-resources-plugin-2.6.pom
Downloaded: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-resources-plugin/2.6/maven-resources-plugin-2.6.pom (8 KB at 2.0 KB/sec)
Downloading: https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-resources-plugin/2.6/maven-resources-plugin-2.6.jar
```

以下信息结尾表示构建成功

```
[INFO] Building jar: C:\Users\YourName\my-app\target\my-app-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 03:50 min
[INFO] Finished at: 2016-12-05T12:28:31+08:00
[INFO] Final Memory: 16M/140M
[INFO] ------------------------------------------------------------------------
```

现在我们可以看到my-app目录下多了个target目录，这个目录包含了编译后的class文件和项目打包后的jar包等

现在我们就可以执行打包后的my-app-1.0-SNAPSHOT.jar中App类中的helloworld程序了

```
java -cp target\my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
```

如果一切正常的话，就会打印出以下语句

```
Hello World!
```

至此，我们的第一个Maven项目构建成功

### Maven项目约定的目录结构

基于约定由于配置的原则，Maven约定了一些标准的目录结构，按照约定的目录结构存放代码和配置文件就可以做到很少的配置实现一些软件开发中的最佳实践。

Maven项目约定的标准目录结构如下

* src/main/java 	Java源代码根目录
* src/main/resources 	Java资源文件目录
* src/main/filters 	Java资源文件的filter文件目录
* src/main/webapp 	JavaWeb程序文件目录，例如WEB-INF目录和jsp、html、js等网页文件目录
* src/test/java 	Java测试源代码根目录
* src/test/resources 	Java测试资源文件目录
* src/test/filters 	Java测试资源文件的filter文件目录
* src/it 	集成测试目录
* src/assembly 	存放使用maven-assembly-plugin插件时存放该插件的描述文件目录
* src/site 	项目站点文件目录
* target  项目构建生成的中间文件和结果文件目录
* LICENSE.txt 	项目的许可证信息
* NOTICE.txt 	项目需要注意的地方或项目依赖的库描述文件
* README.txt 	项目的readme文件
* pom.xml     项目对象模型描述文件

### Maven的生命周期与插件机制

Maven的生命周期执行命令格式：mvn 任务名 

Maven会执行生命周期内的所有在该任务之前的任务。如：mvn clean，会执行pre-clean和clean两个任务。 

Maven有三套独立的生命周期，分别是clean、default、site。

#### clean生命周期

**clean生命周期用于清理target目录，target目录是上一次default生命周期内任意一个任务生成的一些构建文件**

clean的生命周期如下（mvn clean） 

pre-clean -> clean -> post-clean 

pre-clean：执行clean之前需要完成的任务 

clean：清理上一次构建生成的文件 

post-clean：执行clean之后需要立刻完成的任务 

#### default生命周期

**default生命周期用于构建、测试、打包、发布等一系列操作**

default的生命周期如下（这个是开发者用得最多的）
 
compile -> test -> package -> install -> deploy
 
compile：编译项目
 
test：测试项目
 
package：打包项目
 
install：发布项目到本地maven仓库
 
deploy：发布项目到远程maven仓库 

#### site生命周期

**site生命周期用于生成一个网站或PDF文档包括文档、构建报告、项目质量信息等**

site的生命周期如下 

pre-site -> site -> post-site -> site-deploy
 
pre-site：执行site之前需要完成的任务
 
site：生成项目的文档站点
 
post-site：执行site之后需要完成的任务
 
site-deploy：将生成的文档站点发布到远程服务器 

在maven的这三套生命周期中，开发者还可以使用别的插件或者自定义插件，实现在生命周期的任意阶段添加任务 

#### maven的插件机制

Maven本质上就是一个插件执行框架，所有的工作都是通过插件完成的，包括上面提到的Maven三套生命周期

例如前面介绍的三套生命周期中任务绑定的插件如下：

clean任务对应的插件是maven-clean-plugin

compile任务对应的插件是maven-compiler-plugin

test任务对应的插件是maven-surefire-plugin

package任务对应的插件有很多个，根据pom.xml中\<package/\>标签配置的值选择对应的package插件，例如

* jar对应的是maven-jar-plugin
* war对应的是maven-war-plugin
* ear对应的是maven-ear-plugin
* rar对应的是maven-rar-plugin
* ...

install任务对应的插件是maven-install-plugin

deploy任务对应的插件是maven-deploy-plugin

site任务对应的插件是maven-site-plugin

Maven官网[http://maven.apache.org/plugins/index.html](http://maven.apache.org/plugins/index.html)提供了很多可以使用的插件

**有四个插件组**

* apache维护的在maven仓库目录org/apache/maven/plugins下的插件
* codehaus.org维护的在maven仓库目录org/codehaus/mojo下的插件
* 还有一些在code.google.com托管的插件，这个平时不翻墙访问不了，忽略它
* 还有一些项目自己维护相关的maven插件，例如tomcat、jetty插件

**这些插件大体上分为两类**

* 构建插件：在default生命周期某一个任务执行时执行，需要在pom.xml文件中的\<build/\>标签上配置相关的插件配置
* 报告插件：在site生命周期执行时执行，需要在pom.xml文件中的\<reporting/\>标签上配置相关的插件配置


### Maven项目的配置文件pom.xml

POM是项目对象模型（Project Object Model）的缩写，pom.xml是Maven项目的描述文件，是一个标准的基于schema的xml文件，用于描述项目的基本配置信息、构建配置信息、依赖配置信息、环境配置信息等

#### 一个标准的pom.xml配置

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
     
      <!-- The Basics -->
      <groupId>...</groupId>
      <artifactId>...</artifactId>
      <version>...</version>
      <packaging>...</packaging>
      <dependencies>...</dependencies>
      <parent>...</parent>
      <dependencyManagement>...</dependencyManagement>
      <modules>...</modules>
      <properties>...</properties>
     
      <!-- Build Settings -->
      <build>...</build>
      <reporting>...</reporting>
     
      <!-- More Project Information -->
      <name>...</name>
      <description>...</description>
      <url>...</url>
      <inceptionYear>...</inceptionYear>
      <licenses>...</licenses>
      <organization>...</organization>
      <developers>...</developers>
      <contributors>...</contributors>
     
      <!-- Environment Settings -->
      <issueManagement>...</issueManagement>
      <ciManagement>...</ciManagement>
      <mailingLists>...</mailingLists>
      <scm>...</scm>
      <prerequisites>...</prerequisites>
      <repositories>...</repositories>
      <pluginRepositories>...</pluginRepositories>
      <distributionManagement>...</distributionManagement>
      <profiles>...</profiles>
    </project>
```

#### pom.xml最基本的配置

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
     
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>my-project</artifactId>
      <version>1.0</version>
    </project>
```

pom.xml的根元素标签是\<project/\>，其余所有的xml配置标签均在该标签下

\<modelVersion/\>用于配置pom.xml的schema约束文件的版本，有些集成开发环境可能会用到

\<groupId/\>用于描述一个组织或项目的唯一标识，一般使用公司或组织域名，或者域名再加上项目名（即这里的com.mycompany.app）

\<artifactId/\>用于描述一个项目或模块名，这里使用my-app，这需要确保在groupId下是唯一的就可以了

\<version/\>用于描述一个项目或模块的版本号

#### 配置Maven项目或模块的构建包类型

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <packaging>war</packaging>
      ...
    </project>
```

\<packaging/\>的值可以为pom, jar, maven-plugin, ejb, war, ear, rar, par

* pom表示该项目或模块为pom类型项目，用于被其余的项目当做parent项目继承该项目的pom配置
* jar表示该项目或模块为jar项目，构建时会打包成jar包
* maven-plugin表示该项目为maven插件项目
* war表示该项目或模块为war项目，构建时会打包成war包
* ...

#### Maven的坐标

\<groupId/\>、\<artifactId/\>、\<version/\>三个元素用于描述一个Maven项目或模块的坐标，即groupId:artifactId:version，例如org.codehaus.mojo:my-project:1.0

如果包含项目的构建包类型，则坐标为groupId:artifactId:packaging:version，例如org.codehaus.mojo:my-project:war:1.0

有时候你也会看到Maven会打印出五个元素的坐标，groupId:artifactId:packaging:classifier:version，例如org.codehaus.mojo:my-project:war:jdk15:1.0

#### Maven的依赖管理

什么是Maven项目依赖？在传统的Java项目中，假设我们使用到了MySQL的jdbc驱动包，我们就说这个项目依赖MySQL的jdbc驱动包，同理，一个Maven项目依赖另一个Maven项目，就叫Maven项目依赖

Maven的项目依赖是可传递的，即项目A依赖项目B，项目B依赖项目C，则项目A依赖项目C

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <dependencies>
        <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.0</version>
          <type>jar</type>
          <scope>test</scope>
          <optional>true</optional>
        </dependency>
        ...
      </dependencies>
      ...
    </project>
```

\<dependencies/\>标签元素下配置Maven项目的所有依赖包，每一个\<dependency/\>标签配置一个依赖项，上面是示例中依赖了一个junit4.0的jar包，\<dependency/\>使用Maven坐标定位一个依赖项

\<type/\>表示依赖项的包类型，即依赖项的\<packaging/\>的值

\<scope/\>表示一个依赖项的作用域，上面示例中的test表示该依赖项只在测试范围使用，作用域的可选值如下

* compile 该作用域是默认的作用域，不写\<scope/\>的时候默认使用，表示依赖关系仅在编译期间使用
* provided 该作用域表示依赖关系参与编译、测试，但不会参与打包，JDK或容器会提供这些依赖的运行时
* runtime 该作用域表示依赖关系在运行时范围使用，不会参与编译、测试和打包
* test 该作用域表示依赖关系在测试范围使用
* system 该作用域表示不从Maven仓库获取依赖包，而是从本地操作系统获取依赖关系，需要结合\<systemPath/\>标签使用

**\<optional/\>表示可选的依赖。true表示如果一个项目A依赖本项目，本项目依赖junit，则项目A不会依赖junit，false则相反**

**排除依赖**

排除依赖，下面的配置表示该项目或模块依赖了maven-embedder项目，但是排除掉maven-embedder项目依赖的maven-core模块

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <dependencies>
        <dependency>
          <groupId>org.apache.maven</groupId>
          <artifactId>maven-embedder</artifactId>
          <version>2.0</version>
          <exclusions>
            <exclusion>
              <groupId>org.apache.maven</groupId>
              <artifactId>maven-core</artifactId>
            </exclusion>
          </exclusions>
        </dependency>
        ...
      </dependencies>
      ...
    </project>
```

排除所有依赖，下面的配置表示该项目或模块依赖了maven-embedder项目，但是排除掉所有maven-embedder项目依赖的模块

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <dependencies>
        <dependency>
          <groupId>org.apache.maven</groupId>
          <artifactId>maven-embedder</artifactId>
          <version>3.1.0</version>
          <exclusions>
            <exclusion>
              <groupId>*</groupId>
              <artifactId>*</artifactId>
            </exclusion>
          </exclusions>
        </dependency>
        ...
      </dependencies>
      ...
    </project>
```

**Maven的依赖jar或war包查找网站**

* [https://mvnrepository.com](https://mvnrepository.com)
* [https://search.maven.org](https://search.maven.org)
* [https://repository.apache.org](https://repository.apache.org)

#### Maven项目pom.xml配置继承

Maven可以通过类似继承的方式直接继承一个父项目，该项目会继承所有父项目pom.xml的所有配置，当然在子项目中重新配置相同的配置项会覆盖父项目的配置项

子项目会继承父项目以下配置项

* dependencies
* developers and contributors
* plugin lists
* reports lists
* plugin executions with matching ids
* plugin configuration

例如有一个父项目，父项目的\<packaging/\>配置必须是pom

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
     
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>my-parent</artifactId>
      <version>2.0</version>
      <packaging>pom</packaging>
    </project>
```

则子项目如下配置

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
     
      <parent>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>my-parent</artifactId>
        <version>2.0</version>
        <relativePath>../my-parent</relativePath>
      </parent>
     
      <artifactId>my-project</artifactId>
    </project>
```

这里有个\<relativePath/\>标签，表示父项目的相对路径，这项配置是可选的，这项配置要求在本地或远程仓库查找依赖之前先在相对路径查找依赖

父项目pom.xml配置参考

```xml
    <project>
      <modelVersion>4.0.0</modelVersion>
     
      <repositories>
        <repository>
          <id>central</id>
          <name>Central Repository</name>
          <url>http://repo.maven.apache.org/maven2</url>
          <layout>default</layout>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>
      </repositories>
     
      <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <name>Central Repository</name>
          <url>http://repo.maven.apache.org/maven2</url>
          <layout>default</layout>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
          <releases>
            <updatePolicy>never</updatePolicy>
          </releases>
        </pluginRepository>
      </pluginRepositories>
     
      <build>
        <directory>${project.basedir}/target</directory>
        <outputDirectory>${project.build.directory}/classes</outputDirectory>
        <finalName>${project.artifactId}-${project.version}</finalName>
        <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
        <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
        <scriptSourceDirectory>src/main/scripts</scriptSourceDirectory>
        <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
        <resources>
          <resource>
            <directory>${project.basedir}/src/main/resources</directory>
          </resource>
        </resources>
        <testResources>
          <testResource>
            <directory>${project.basedir}/src/test/resources</directory>
          </testResource>
        </testResources>
        <pluginManagement>
          <!-- NOTE: These plugins will be removed from future versions of the super POM -->
          <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
          <plugins>
            <plugin>
              <artifactId>maven-antrun-plugin</artifactId>
              <version>1.3</version>
            </plugin>
            <plugin>
              <artifactId>maven-assembly-plugin</artifactId>
              <version>2.2-beta-5</version>
            </plugin>
            <plugin>
              <artifactId>maven-dependency-plugin</artifactId>
              <version>2.1</version>
            </plugin>
            <plugin>
              <artifactId>maven-release-plugin</artifactId>
              <version>2.0</version>
            </plugin>
          </plugins>
        </pluginManagement>
      </build>
     
      <reporting>
        <outputDirectory>${project.build.directory}/site</outputDirectory>
      </reporting>
     
      <profiles>
        <!-- NOTE: The release profile will be removed from future versions of the super POM -->
        <profile>
          <id>release-profile</id>
     
          <activation>
            <property>
              <name>performRelease</name>
              <value>true</value>
            </property>
          </activation>
     
          <build>
            <plugins>
              <plugin>
                <inherited>true</inherited>
                <artifactId>maven-source-plugin</artifactId>
                <executions>
                  <execution>
                    <id>attach-sources</id>
                    <goals>
                      <goal>jar</goal>
                    </goals>
                  </execution>
                </executions>
              </plugin>
              <plugin>
                <inherited>true</inherited>
                <artifactId>maven-javadoc-plugin</artifactId>
                <executions>
                  <execution>
                    <id>attach-javadocs</id>
                    <goals>
                      <goal>jar</goal>
                    </goals>
                  </execution>
                </executions>
              </plugin>
              <plugin>
                <inherited>true</inherited>
                <artifactId>maven-deploy-plugin</artifactId>
                <configuration>
                  <updateReleaseInfo>true</updateReleaseInfo>
                </configuration>
              </plugin>
            </plugins>
          </build>
        </profile>
      </profiles>
     
    </project>
```

子项目的pom.xml在最终打包的时候会与parent的pom.xml聚合在一个pom.xml文件中，可以通过命令行查看当前Maven项目的有效配置

```
mvn help:effective-pom
```

#### Maven的\<dependencyManagement/\>标签

\<dependencyManagement/\>标签也是依赖管理标签，可以在\<dependencyManagement/\>中配置\<dependencies/\>标签，而原来的\<dependencies/\>这只需要配置\<groupId/\>和\<artifactId/\>即可，当然在\<dependencies/\>标签中也可覆盖\<dependencyManagement/\>标签的配置

\<dependencyManagement/\>标签一般在父项目中使用，而\<dependencies/\>在子项目中使用，当有多个子项目时，依赖的具体信息则全部配置父项目中，当需要对某一个公共依赖修改时，则只需要修改父项目一处即可

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <dependencyManagement>
          <dependencies>
            <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.0</version>
              <type>jar</type>
              <scope>test</scope>
              <optional>true</optional>
            </dependency>
            ...
          </dependencies>
      </dependencyManagement>
      ...
      <dependencies>
          <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
          </dependency>
          ...
      </dependencies>
      ...
    </project>
```

#### Maven多模块项目配置

Maven多模块项目的目录结构一般如下

```
    my-parent
    |-- pom.xml
    `-- my-project
        |-- pom.xml
    `-- another-project
        |-- pom.xml
```

每一个模块包括parent模块的标准目录结构则跟之前单模块Maven项目的约定一致

在parent项目中配置\<modules\>标签则可以通过parent项目管理多个子项目的构建

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
     
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>my-parent</artifactId>
      <version>2.0</version>
      <packaging>pom</packaging>
     
      <modules>
        <module>my-project</module>
        <module>another-project</module>
      </modules>
    </project>
```

#### properties配置

pom.xml中的properties就相当于变量定义，在需要的地方通过${propertyName}引用

一般依赖关系的版本集中在\<properties/\>配置，例如

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <properties>
        <junit.version>4.0</junit.version>
      </properties>
      <dependencies>
        <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>${junit.version}</version>
          <type>jar</type>
          <scope>test</scope>
          <optional>true</optional>
        </dependency>
        ...
      </dependencies>
      ...
    </project>
```

#### Maven的构建信息配置

Maven的构建信息配置是在\<build/\>标签下配置的

在\<build/\>标签下可以对已有的默认约定配置进行配置和插件的配置，例如源码目录配置、资源文件配置、filter配置等

与default生命周期绑定

**对于源码目录、测试源码目录、构建输出的目录和测试目录进行配置，以下是Maven默认的配置**

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <build>
        ...
        <sourceDirectory>${basedir}/src/main/java</sourceDirectory>
        <scriptSourceDirectory>${basedir}/src/main/scripts</scriptSourceDirectory>
        <testSourceDirectory>${basedir}/src/test/ava</testSourceDirectory>
        <outputDirectory>${basedir}/target/classes</outputDirectory>
        <testOutputDirectory>${basedir}/target/test-classes</testOutputDirectory>
        ...
      </build>
    </project>
```

**对资源文件和测试资源文件进行配置**

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <build>
        ...
        <resources>
          <resource>
            <targetPath>META-INF/plexus</targetPath>
            <filtering>false</filtering>
            <directory>${basedir}/src/main/plexus</directory>
            <includes>
              <include>configuration.xml</include>
            </includes>
            <excludes>
              <exclude>**/*.properties</exclude>
            </excludes>
          </resource>
        </resources>
        <testResources>
          ...
        </testResources>
        ...
      </build>
    </project>
```

**使用\<filter/\>资源文件进行过滤**

在\<resource/\>标签下配置\<filtering/\>

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          http://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <build>
          <filters>
            <filter>src/main/filters/filter.properties</filter>
          </filters>
          <resources>
            <resource>
              <directory>src/main/resources</directory>
              <filtering>true</filtering>
            </resource>
          </resources>
        </build>
    </project>
```

例如/src/main/resources目录下有一个application.properties配置文件，配置的值使用${propertyValue}的形式占位

```properties
    # application.properties
    application.name=${application.name}
    application.version=${application.name}
```

然后创建一个文件夹/src/main/filters，并在该文件夹下创建一个filter.properties文件，配置如下

```properties
    filter.properties
    application.name=my-app
    application.version=1.0
```

在项目构建时，就会将application.properties文件中的占位符自动改为filter.properties文件中对应的值。这在与profile结合使用时比较有用，可以使开发、测试、部署使用不同的配置值

**插件配置**

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <build>
        ...
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>2.6</version>
            <extensions>false</extensions>
            <inherited>true</inherited>
            <configuration>
              <classifier>test</classifier>
            </configuration>
            <dependencies>...</dependencies>
            <executions>...</executions>
          </plugin>
        </plugins>
      </build>
    </project>
```

#### 插件\<pluginManagement/\>配置

\<pluginManagement/\>与\<dependencyManagement/\>类似，配好之后，在\<plugin/\>中只需要配置\<groupId/\>和\<artifactId/\>，其它的配置项同样是可以在\<plugin/\>中覆盖的

\<pluginManagement/\>一般在parent的pom.xml中配置

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <build>
        ...
        <pluginManagement>
          <plugins>
            <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-jar-plugin</artifactId>
              <version>2.6</version>
              <executions>
                <execution>
                  <id>pre-process-classes</id>
                  <phase>compile</phase>
                  <goals>
                    <goal>jar</goal>
                  </goals>
                  <configuration>
                    <classifier>pre-process</classifier>
                  </configuration>
                </execution>
              </executions>
            </plugin>
          </plugins>
        </pluginManagement>
        ...
      </build>
    </project>
```

在\<plugins/\>中使用

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <build>
        ...
        <plugins>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
          </plugin>
        </plugins>
        ...
      </build>
    </project>
```

#### Maven的报告站点生成配置

Maven的报告站点生成配置在\<reporting/\>标签中配置，与\<build/\>标签的配置方式基本一致，主要是配置site输出目录和插件配置，同样可以配置\<pluginManagement/\>标签

与site生命周期绑定

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <reporting>
        <outputDirectory>${basedir}/target/site</outputDirectory>
        <plugins>
          <plugin>
            <artifactId>maven-project-info-reports-plugin</artifactId>
            <version>2.0.1</version>
            <reportSets>
              <reportSet></reportSet>
            </reportSets>
          </plugin>
        </plugins>
      </reporting>
      ...
    </project>
```

#### Maven仓库管理

在pom.xml中可以配置的仓库分为以下几种

* 查找依赖关系的仓库，配置在\<repositories/\>
* 查找插件依赖的仓库，配置在\<pluginRepositories\>
* deploy部署的仓库，配置在\<distributionManagement/\>

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <repositories>
        <repository>
          <releases>
            <enabled>false</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
          </snapshots>
          <id>codehausSnapshots</id>
          <name>Codehaus Snapshots</name>
          <url>http://snapshots.maven.codehaus.org/maven2</url>
          <layout>default</layout>
        </repository>
      </repositories>
      <pluginRepositories>
        ...
      </pluginRepositories>
      ...
    </project>
```

**将非Maven管理的jar发布到本地仓库**

```
mvn install:install-file -Dfile=non-maven-proj.jar -DgroupId=some.group -DartifactId=non-maven-proj -Dversion=1 -Dpackaging=jar
```

**将非Maven管理的jar发布到远程仓库**

```
mvn deploy:deploy-file -Dfile=non-maven-proj.jar -DgroupId=some.group -DartifactId=non-maven-proj -Dversion=1 -Dpackaging=jar
```

#### 发布管理

发布管理在\<distributionManagement/\>配置，可以配置如下信息

* 项目下载地址\<downloadUrl/\>
* 项目状态\<status/\>
* deploy发布仓库\<repository/\>和\<snapshotRepository/\>
* 项目报告信息\<site/\>
* 项目搬家到新地址了\<relocation/\>

项目下载地址\<downloadUrl\>和项目状态\<status/\>

\<status/\>可选的值有none、converted、partner、deployed、verified

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <distributionManagement>
        ...
        <downloadUrl>http://mojo.codehaus.org/my-project</downloadUrl>
        <status>deployed</status>
      </distributionManagement>
      ...
    </project>
```

deploy发布仓库\<repository/\>和\<snapshotRepository/\>

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <distributionManagement>
        <repository>
          <uniqueVersion>false</uniqueVersion>
          <id>corp1</id>
          <name>Corporate Repository</name>
          <url>scp://repo/maven2</url>
          <layout>default</layout>
        </repository>
        <snapshotRepository>
          <uniqueVersion>true</uniqueVersion>
          <id>propSnap</id>
          <name>Propellors Snapshots</name>
          <url>sftp://propellers.net/maven</url>
          <layout>legacy</layout>
        </snapshotRepository>
        ...
      </distributionManagement>
      ...
    </project>
```

项目报告信息\<site/\>

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <distributionManagement>
        ...
        <site>
          <id>mojo.website</id>
          <name>Mojo Website</name>
          <url>scp://beaver.codehaus.org/home/projects/mojo/public_html/</url>
        </site>
        ...
      </distributionManagement>
      ...
    </project>
```

项目搬家到新地址了\<relocation/\>

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <distributionManagement>
        ...
        <relocation>
          <groupId>org.apache</groupId>
          <artifactId>my-project</artifactId>
          <version>1.0</version>
          <message>We have moved the Project under Apache</message>
        </relocation>
        ...
      </distributionManagement>
      ...
    </project>
```

#### 持续集成管理

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <ciManagement>
        <system>continuum</system>
        <url>http://127.0.0.1:8080/continuum</url>
        <notifiers>
          <notifier>
            <type>mail</type>
            <sendOnError>true</sendOnError>
            <sendOnFailure>true</sendOnFailure>
            <sendOnSuccess>false</sendOnSuccess>
            <sendOnWarning>false</sendOnWarning>
            <configuration><address>continuum@127.0.0.1</address></configuration>
          </notifier>
        </notifiers>
      </ciManagement>
      ...
    </project>
```

#### Maven的profile配置

profile是Maven对不同环境提供不同配置的一种方式，在传统的项目开发过程中，我们可能需要经历dev（开发）、test（测试）、sit（集成测试）、uat（用户验收测试）、prod（产品发布）等等阶段，对于每一个阶段我们可能有某些配置希望是不一样的，而profile提供了这个配置方式

在Maven中，profile基本上可以配置前面介绍的所有pom.xml配置信息，典型的profile配置片段如下

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <profiles>
        <profile>
          <id>dev</id>
          <activation>...</activation>
          <properties>...</properties>
          <build>...</build>
          <modules>...</modules>
          <repositories>...</repositories>
          <pluginRepositories>...</pluginRepositories>
          <dependencies>...</dependencies>
          <reporting>...</reporting>
          <dependencyManagement>...</dependencyManagement>
          <distributionManagement>...</distributionManagement>
        </profile>
        <profile>
          <id>test</id>
          <activation>...</activation>
          <properties>...</properties>
          <build>...</build>
          <modules>...</modules>
          <repositories>...</repositories>
          <pluginRepositories>...</pluginRepositories>
          <dependencies>...</dependencies>
          <reporting>...</reporting>
          <dependencyManagement>...</dependencyManagement>
          <distributionManagement>...</distributionManagement>
        </profile>
        ...
      </profiles>
    </project>
```

其中里面的\<activation/\>是可选的，例如可以默认激活一个profile，\<activeByDefault\>true\</activeByDefault\>

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <profiles>
        <profile>
          <id>dev</id>
          <activation>
            <activeByDefault>true</activeByDefault>
          </activation>
        </profile>
        ...
      </profiles>
    </project>
```

在命令行构建项目是可以加一个-P参数指定需要的profile

```
mvn package -P test
```

查看当前使用的profile

```
mvn help:active-profiles
```

#### 更多pom.xml配置信息

以下信息一般开源项目用的多一些，我们平时用的很少

项目许可证配置

```xml
    <licenses>
      <license>
        <name>Apache License, Version 2.0</name>
        <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        <distribution>repo</distribution>
        <comments>A business-friendly OSS license</comments>
      </license>
    </licenses>
```

项目所有组织配置

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <organization>
        <name>Codehaus Mojo</name>
        <url>http://mojo.codehaus.org</url>
      </organization>
    </project>
```

项目开发者列表配置

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <developers>
        <developer>
          <id>jdoe</id>
          <name>John Doe</name>
          <email>jdoe@example.com<email>
          <url>http://www.example.com/jdoe</url>
          <organization>ACME</organization>
          <organizationUrl>http://www.example.com</organizationUrl>
          <roles>
            <role>architect</role>
            <role>developer</role>
          </roles>
          <timezone>America/New_York</timezone>
          <properties>
            <picUrl>http://www.example.com/jdoe/pic</picUrl>
          </properties>
        </developer>
      </developers>
      ...
    </project>
```

项目贡献者列表

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <contributors>
        <contributor>
          <name>Noelle</name>
          <email>some.name@gmail.com</email>
          <url>http://noellemarie.com</url>
          <organization>Noelle Marie</organization>
          <organizationUrl>http://noellemarie.com</organizationUrl>
          <roles>
            <role>tester</role>
          </roles>
          <timezone>America/Vancouver</timezone>
          <properties>
            <gtalk>some.name@gmail.com</gtalk>
          </properties>
        </contributor>
      </contributors>
      ...
    </project>
```

Issue管理

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <issueManagement>
        <system>Bugzilla</system>
        <url>http://127.0.0.1/bugzilla/</url>
      </issueManagement>
      ...
    </project>
```

邮件列表管理

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <mailingLists>
        <mailingList>
          <name>User List</name>
          <subscribe>user-subscribe@127.0.0.1</subscribe>
          <unsubscribe>user-unsubscribe@127.0.0.1</unsubscribe>
          <post>user@127.0.0.1</post>
          <archive>http://127.0.0.1/user/</archive>
          <otherArchives>
            <otherArchive>http://base.google.com/base/1/127.0.0.1</otherArchive>
          </otherArchives>
        </mailingList>
      </mailingLists>
      ...
    </project>
```

软件配置管理SCM，可以配置版本控制，比如SVN

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <scm>
        <connection>scm:svn:http://127.0.0.1/svn/my-project</connection>
        <developerConnection>scm:svn:https://127.0.0.1/svn/my-project</developerConnection>
        <tag>HEAD</tag>
        <url>http://127.0.0.1/websvn/my-project</url>
      </scm>
      ...
    </project>
```

最低版本配置要求\<prerequisites/\>，例如配置最低maven版本为2.0.6

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <prerequisites>
        <maven>2.0.6</maven>
      </prerequisites>
      ...
    </project>
```

更详细信息参考[http://maven.apache.org/pom.html](http://maven.apache.org/pom.html)


### Maven的配置文件settings.xml

settings.xml是对Maven本身的配置，这里面的配置相对pom.xml就少很多了，而且一般项目的配置也不应该配置在这个地方

settings.xml结构如下

```xml
        <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
          <localRepository/>
          <interactiveMode/>
          <usePluginRegistry/>
          <offline/>
          <pluginGroups/>
          <servers/>
          <mirrors/>
          <proxies/>
          <profiles/>
          <activeProfiles/>
        </settings>
```

一般我们只需要关注\<localRepository/\>、\<servers/\>和\<mirrors/\>就行了

#### 基本信息配置

```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
      <localRepository>${user.home}/.m2/repository</localRepository>
      <interactiveMode>true</interactiveMode>
      <usePluginRegistry>false</usePluginRegistry>
      <offline>false</offline>
      ...
    </settings>
```

\<localRepository/\>配置本地仓库的路径，也就是Maven管理依赖的jar或war存放路径

\<interactiveMode/\>表示使用mvn命令是是否使用交互式模式

\<usePluginRegistry/\>默认为false，maven2.0以后这项配置相当于是弃用了

\<offline/\>构建系统是否使用离线模式，即不从网络仓库中获取Maven依赖关系

#### 插件组\<pluginGroups/\>

前面介绍Maven的生命周期和插件机制时提到Maven默认有四类插件组，如果是自己编写的插件或者不在默认插件组中的插件

我们在使用时需要使用插件的完整坐标，例如

```
mvn org.mortbay.jetty:jetty-maven-plugin:run
```

在插件组中配置jetty插件

```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
      ...
      <pluginGroups>
        <pluginGroup>org.mortbay.jetty</pluginGroup>
      </pluginGroups>
      ...
    </settings>
```

那么我们在使用的时候，就可以执行以下命令了

```
mvn jetty:run
```

当然jetty本身是在maven默认的插件组里面的，所以就不需要配置插件组了

#### Maven的\<Server/\>

Maven的依赖关系下载、插件下载、项目发布的仓库都是都是在pom.xml中配置的，但是有些信息比如用户名和密码之类的信息不应该配置在pom.xml中，而是应该配置在settings.xml中

```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
      ...
      <servers>
        <server>
          <id>server001</id>
          <username>my_login</username>
          <password>my_password</password>
          <privateKey>${user.home}/.ssh/id_dsa</privateKey>
          <passphrase>some_passphrase</passphrase>
          <filePermissions>664</filePermissions>
          <directoryPermissions>775</directoryPermissions>
          <configuration></configuration>
        </server>
      </servers>
      ...
    </settings>
```

那么在pom.xml中我们需要在\<repository/\>标签内配置一个\<id/\>，这个\<id/\>的值需要与上面的\<server/\>标签下的\<id/\>的值保持一致

```xml
    <project xmlns="http://maven.apache.org/POM/4.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                          https://maven.apache.org/xsd/maven-4.0.0.xsd">
      ...
      <repositories>
        <repository>
          <id>server001</id>
          ...
         </repository>
          ...
      </repositories>
      <pluginRepositories>
        ...
      </pluginRepositories>
      ...
    </project>
```

#### Maven的仓库镜像配置

这里的仓库镜像指的是下载依赖关系和插件的仓库镜像

中央仓库的地址是https://repo.maven.apache.org/maven2

默认情况下，Maven有个中央仓库，中央仓库其实也是一个镜像，如果我们把在\<mirror/\>标签下配置一个\<mirrorOf/\>为central的值，那么我们就下载依赖和插件时就不会从中央仓库下载，而是从配置的镜像的\<url/\>中去下载

**使用单个mirror**

\<mirrorOf/\>为central或*

```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
      ...
      <mirrors>
        <mirror>
          <id>planetmirror.com</id>
          <name>PlanetMirror Australia</name>
          <url>http://downloads.planetmirror.com/pub/maven2</url>
          <mirrorOf>central</mirrorOf>
        </mirror>
      </mirrors>
      ...
    </settings>
```

\<id/\>和\<name/\>可随意配置

\<url/\>是就是镜像的地址，可以结合公司私服使用

\<mirrorOf/\>这里配置仓库的镜像，例如上面的central表示是中央仓库的镜像，*表示所有仓库镜像, \<mirrorOf/\>可以使用以下值

* \* = 表示所有仓库的镜像
* external:* = 表示所有仓库的镜像但是排除本地仓库
* repo,repo1 = 仓库repo,repo1的镜像
* \*,!repo1 = 所有仓库镜像排除仓库repo1

**使用多个mirror**

```xml
    <settings>
      ...
      <mirrors>
        <mirror>
          <id>internal-repository</id>
          <name>Maven Repository Manager running on repo.mycompany.com</name>
          <url>http://repo.mycompany.com/proxy</url>
          <mirrorOf>external:*,!foo</mirrorOf>
        </mirror>
        <mirror>
          <id>foo-repository</id>
          <name>Foo</name>
          <url>http://repo.mycompany.com/foo</url>
          <mirrorOf>foo</mirrorOf>
        </mirror>
      </mirrors>
      ...
    </settings>
```

#### Maven代理

有时候因为网络问题我们可能需要配置一下代理来访问网络仓库，在\<proxies/\>中配置

```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
      ...
      <proxies>
        <proxy>
          <id>myproxy</id>
          <active>true</active>
          <protocol>http</protocol>
          <host>proxy.somewhere.com</host>
          <port>8080</port>
          <username>proxyuser</username>
          <password>somepassword</password>
          <nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>
        </proxy>
      </proxies>
      ...
    </settings>
```

#### \<profiles/\>和\<activeProfiles/\>

\<profiles/\>和\<activeProfiles/\>的配置与pom.xml类似，一般在pom.xml中配置

```xml
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
      <profiles>
          <profile>
            <id>dev</id>
            <properties>...</properties>
            ...
          </profile>
          <profile>
            <id>test</id>
            <properties>...</properties>
            ...
          </profile>
          ...
    </profiles>
      ...
      <activeProfiles>
        <activeProfile>dev</activeProfile>
      </activeProfiles>
    </settings>
```

查看当前激活的profile

```
mvn help:active-profiles
```

查看有效的settings.xml配置

```
mvn help:effective-settings
```

更详细信息参考[http://maven.apache.org/settings.html](http://maven.apache.org/settings.html)

### Maven多模块项目及最佳实践

前面已经做了第一个Maven项目，现在我们使用Maven创建一个多模块项目

在第一个Maven项目中，我们使用了maven-archetype-plugin插件生成Maven项目的原型骨架，现在我们同样用这种方式生成项目的原型骨架

首先我们先生成一个pom项目，作为项目的根项目，也是这个多模块项目的parent项目

```
mvn archetype:generate -DgroupId=org.jisonami -DartifactId=app -Dversion=1.0 -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DarchetypeCatalog=local
```

生成的是jar项目，我们把其它的文件夹都删了，保留pom.xml文件，然后把pom.xml里面的\<packaging/\>的值改为pom

进入app目录

```
cd app
```

创建一个子项目app_common

```
mvn archetype:generate -DgroupId=org.jisonami -DartifactId=app_common -Dversion=1.0 -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DarchetypeCatalog=local
```

创建一个子项目app_web

```
mvn archetype:generate -DgroupId=org.jisonami -DartifactId=app_web -Dversion=1.0 -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false -DarchetypeCatalog=local
```

现在一个包含两个子项目的app项目就生成了

配置app目录下的pom.xml文件如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.jisonami</groupId>
	<artifactId>app</artifactId>
	<packaging>pom</packaging>
	<version>1.0</version>
	<name>app</name>
	<url>http://app.jisonami.org</url>

	<!-- 子模块配置 -->
	<modules>
		<module>app_common</module>
		<module>app_web</module>
	</modules>

	<!-- 属性配置 -->
	<properties>
		<java.version>1.8</java.version>
		<springframework.version>4.2.0.RELEASE</springframework.version>
		<junit.version>4.11</junit.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<fastjson.version>1.1.31</fastjson.version>
		<json-path.version>0.8.1</json-path.version>
		<jackson.version>1.8.4</jackson.version>
		<mybatis.version>3.2.8</mybatis.version>
		<mybatis-spring.version>1.2.2</mybatis-spring.version>
		<slf4j.version>1.7.7</slf4j.version>
		<logback.version>1.1.2</logback.version>
		<logback-ext-spring.version>0.1.1</logback-ext-spring.version>
		<mysql.version>5.1.38</mysql.version>
		<java.servlet.api.version>3.0.1</java.servlet.api.version>
	</properties>

	<!-- 依赖管理配置 -->
	<dependencyManagement>
		<dependencies>

			<!-- servlet-api -->
			<dependency>
				<groupId>javax.servlet</groupId>
				<artifactId>javax.servlet-api</artifactId>
				<version>${java.servlet.api.version}</version>
				<scope>provided</scope>
			</dependency>

			<!-- logback日志配置开始 -->
			<dependency>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-api</artifactId>
				<version>${slf4j.version}</version>
			</dependency>

			<dependency>
				<groupId>ch.qos.logback</groupId>
				<artifactId>logback-core</artifactId>
				<version>${logback.version}</version>
			</dependency>

			<dependency>
				<groupId>ch.qos.logback</groupId>
				<artifactId>logback-classic</artifactId>
				<version>${logback.version}</version>
			</dependency>

			<dependency>
				<groupId>org.logback-extensions</groupId>
				<artifactId>logback-ext-spring</artifactId>
				<version>${logback-ext-spring.version}</version>
			</dependency>

			<!-- mysql jdbc driver -->
			<dependency>
				<groupId>mysql</groupId>
				<artifactId>mysql-connector-java</artifactId>
				<version>${mysql.version}</version>
			</dependency>

			<!--=========================== IBatis ====================================================================== -->
			<dependency>
				<groupId>org.mybatis</groupId>
				<artifactId>mybatis</artifactId>
				<version>${mybatis.version}</version>
			</dependency>
			<dependency>
				<groupId>org.mybatis</groupId>
				<artifactId>mybatis-spring</artifactId>
				<version>${mybatis-spring.version}</version>
			</dependency>
			<!--=========================== Spring ===================================================================== -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-beans</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-test</artifactId>
				<version>${springframework.version}</version>
				<scope>test</scope>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-tx</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-web</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-context-support</artifactId>
				<version>${springframework.version}</version>
			</dependency>

			<!--=========================== Apache Commons ============================================================== -->
			<dependency>
				<groupId>org.apache.commons</groupId>
				<artifactId>commons-lang3</artifactId>
				<version>${commons-lang3.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-beanutils</groupId>
				<artifactId>commons-beanutils</artifactId>
				<version>${commons-beanutils.version}</version>
			</dependency>

			<dependency>
				<groupId>commons-digester</groupId>
				<artifactId>commons-digester</artifactId>
				<version>${commons-digester.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-dbcp</groupId>
				<artifactId>commons-dbcp</artifactId>
				<version>${commons-dbcp.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-fileupload</groupId>
				<artifactId>commons-fileupload</artifactId>
				<version>${commons-fileupload.version}</version>
			</dependency>

			<!--=========================== JUnit ======================================================================= -->
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>${junit.version}</version>
				<scope>test</scope>
			</dependency>

			<!--=========================== JSON ======================================================================== -->
			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>fastjson</artifactId>
				<version>${fastjson.version}</version>
			</dependency>
			<dependency>
				<groupId>com.jayway.jsonpath</groupId>
				<artifactId>json-path</artifactId>
				<version>${json-path.version}</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>org.codehaus.jackson</groupId>
				<artifactId>jackson-core-asl</artifactId>
				<version>${jackson.version}</version>
			</dependency>
			<dependency>
				<groupId>org.codehaus.jackson</groupId>
				<artifactId>jackson-mapper-asl</artifactId>
				<version>${jackson.version}</version>
			</dependency>

		</dependencies>
	</dependencyManagement>

	<!-- 公共依赖配置 -->
	<dependencies>
		<!--=========================== JUnit ======================================================================= -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
		</dependency>

		<!--=========================== JSON ======================================================================== -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
		</dependency>
		<dependency>
			<groupId>com.jayway.jsonpath</groupId>
			<artifactId>json-path</artifactId>
		</dependency>
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-core-asl</artifactId>
		</dependency>
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
		</dependency>
	</dependencies>

	<!-- 依赖仓库配置 -->
	<repositories>
		<repository>
			<id>jisonami-public</id>
			<name>jisonami-public</name>
			<url>http://org.jisonami:8081/nexus/content/groups/public/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>jisonami-release</id>
			<name>jisonami-release</name>
			<url>http://org.jisonami:8081/nexus/content/repositories/releases/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>jisonami-thirdparty</id>
			<name>jisonami-thirdparty</name>
			<url>http://org.jisonami:8081/nexus/content/repositories/thirdparty/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/libs-milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

	<!-- 插件依赖仓库配置 -->
	<pluginRepositories>
		<pluginRepository>
			<id>jisonami-public</id>
			<name>jisonami-public</name>
			<url>http://org.jisonami:8081/nexus/content/groups/public/</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</pluginRepository>
	</pluginRepositories>

	<!-- 项目发布仓库配置 -->
	<distributionManagement>
		<repository>
			<id>releases</id>
			<name>Nexus Releases</name>
			<url>http://org.jisonami:8081/nexus/content/repositories/releases</url>
		</repository>
		<snapshotRepository>
			<id>snapshots</id>
			<name>Nexus Repository</name>
			<url>http://org.jisonami:8081/nexus/content/repositories/snapshots</url>
		</snapshotRepository>
	</distributionManagement>

	<!-- 构建配置 -->
	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<artifactId>maven-compiler-plugin</artifactId>
					<configuration>
						<source>${java.version}</source>
						<target>${java.version}</target>
						<encoding>${project.build.sourceEncoding}</encoding>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>

</project>
```

配置app_common目录下的pom.xml文件如下

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.jisonami</groupId>
    <artifactId>app</artifactId>
    <version>1.0</version>
  </parent>
  <groupId>org.jisonami</groupId>
  <artifactId>app_common</artifactId>
  <version>1.0</version>
  <name>app_common</name>
  <url>http://app.jisonami.org</url>
  
  <!-- 依赖配置 -->
  <dependencies>
  
    <!-- mysql jdbc driver -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
	</dependency>
	
	<!--=========================== IBatis ====================================================================== -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
	</dependency>
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis-spring</artifactId>
	</dependency>
	
	<!--=========================== Spring ===================================================================== -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-beans</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-test</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-tx</artifactId>
	</dependency>
	
  </dependencies>

	<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

配置app_web目录下的pom.xml文件如下

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.jisonami</groupId>
        <artifactId>app</artifactId>
        <version>1.0</version>
    </parent>
    <groupId>org.jisonami</groupId>
    <artifactId>app_web</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>
    <name>app_web Maven Webapp</name>
    <url>http://app.jisonami.org</url>

    <!-- 依赖配置 -->
    <dependencies>
    
    	<!-- servlet-api -->
    	<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
		</dependency>
    
        <!--=========================== Spring ===================================================================== -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
        </dependency>
    </dependencies>

	<profiles>
        <profile>
            <id>dev</id>
            <properties>
                <env>dev</env>
            </properties>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <env>test</env>
            </properties>
        </profile>
        <profile>
            <id>sit</id>
            <properties>
                <env>sit</env>
            </properties>
        </profile>
        <profile>
            <id>uat</id>
            <properties>
                <env>uat</env>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <env>prod</env>
            </properties>
        </profile>
    </profiles>
	
    <build>
        <finalName>app_web</finalName>

        <filters>
            <filter>src/main/filters/${env}/datasource.properties</filter>
        </filters>

		<resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
		
        <plugins>

			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
			</plugin>
		
            <!-- tomcat7插件 -->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <port>8888</port>
                    <uriEncoding>${project.build.sourceEncoding}</uriEncoding>
                    <server>tomcat7</server>
                </configuration>
            </plugin>

            <!-- jetty插件 -->
            <plugin>
                <groupId>org.mortbay.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>7.6.14.v20131031</version>
                <configuration>
                    <connectors>
                        <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
                            <port>8181</port>
                        </connector>
                    </connectors>
                    <webAppConfig>
                        <contextPath>/${project.artifactId}</contextPath>
                    </webAppConfig>
                    <systemProperties>
                        <systemProperty>
                            <name>org.mortbay.util.URI.charset</name>
                            <value>${project.build.sourceEncoding}</value>
                        </systemProperty>
                    </systemProperties>
                    <stopKey/>
                    <stopPort/>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

在app_web模块的/src/main/resources添加一个datasource.properties，内容如下

```properties
# datasource config
app.datasource.url=${app.datasource.url}
app.datasource.driverClassName=com.mysql.jdbc.Driver
app.datasource.username=${app.datasource.username}
app.datasource.password=${app.datasource.password}
```

在app_web模块下添加/src/main/filters目录，分别添加dev、test、sit、uat、prod目录，并各添加一个datasource.properties文件

dev目录下的datasource.properties文件内容如下

```properties
# datasource config
app.datasource.url=jdbc:mysql://dev.jisonami.org:3306:app
app.datasource.username=dev
app.datasource.password=dev
```

test目录下的properties文件内容如下

```properties
# datasource config
app.datasource.url=jdbc:mysql://test.jisonami.org:3306:app
app.datasource.username=test
app.datasource.password=test
```

sit目录下的datasource.properties文件内容如下

```properties
# datasource config
app.datasource.url=jdbc:mysql://sit.jisonami.org:3306:app
app.datasource.username=sit
app.datasource.password=sit
```

uat目录下的datasource.properties文件内容如下

```properties
# datasource config
app.datasource.url=jdbc:mysql://uat.jisonami.org:3306:app
app.datasource.username=uat
app.datasource.password=uat
```

prod目录下的datasource.properties文件内容如下

```properties
# datasource config
app.datasource.url=jdbc:mysql://prod.jisonami.org:3306:app
app.datasource.username=prod
app.datasource.password=prod
```

命令行输入以下命令查看目录结构

```
tree /f
```

现在app项目的目录结构如下

```
─app
    │  pom.xml
    │
    ├─app_common
    │  │  pom.xml
    │  │
    │  └─src
    │      ├─main
    │      │  └─java
    │      │      └─jar
    │      │              App.java
    │      │
    │      └─test
    │          └─java
    │              └─jar
    │                      AppTest.java
    │
    └─app_web
        │  pom.xml
        │
        └─src
            └─main
                ├─filters
                │  ├─dev
                │  │      datasource.properties
                │  │
                │  ├─prod
                │  │      datasource.properties
                │  │
                │  ├─sit
                │  │      datasource.properties
                │  │
                │  ├─test
                │  │      datasource.properties
                │  │
                │  └─uat
                │          datasource.properties
                │
                ├─resources
                │      datasource.properties
                │
                └─webapp
                    │  index.jsp
                    │
                    └─WEB-INF
                            web.xml
```

进入app_web目录

```
cd app_web
```

使用tomcat7插件运行app_web项目

```
mvn clean tomcat7:run
```

使用里浏览器访问localhost:8888/app_web/

![app_web_tomcat7-run.png](/images/Maven/MavenUsageSummary/app_web_tomcat7-run.png)

使用jetty插件运行app_web项目

```
mvn clean jetty:run
```

使用里浏览器访问localhost:8888/app_web/

![app_web_jetty-run.png](/images/Maven/MavenUsageSummary/app_web_jetty-run.png)

先退回到app目录, 即上一个目录

```
cd ..
```

对整个项目进行打包

```
mvn clean package
```

构建完成后，打开app\app_web\target\app_web\WEB-INF\classes的datasource.properties文件，发现里面的配置确实被src/main/filters/datasource.properties的配置替代了，我们配置的默认的profile生效了

```
# datasource config
app.datasource.url=jdbc:mysql://dev.jisonami.org:3306:app
app.datasource.driverClassName=com.mysql.jdbc.Driver
app.datasource.username=dev
app.datasource.password=dev
```

对整个项目进行打包，指定profile为test

```
mvn clean package -P test
```

构建完成后，打开app\app_web\target\app_web\WEB-INF\classes的datasource.properties文件，发现里面的配置确实被src/main/filters/datasource.properties的配置替代了，我们配置的test的profile生效了

```
# datasource config
app.datasource.url=jdbc:mysql://test.jisonami.org:3306:app
app.datasource.driverClassName=com.mysql.jdbc.Driver
app.datasource.username=test
app.datasource.password=test
```

### 在集成开发环境中使用Maven

#### Eclipse中使用Maven

##### 环境配置

配置截图步骤如下

![Window_Preferences.png](/images/Maven/MavenUsageSummary/Eclipse/Window_Preferences.png)

配置使用自己下载Maven而不是Eclipse自带的Maven，避免Eclipse自带的Maven因版本不一致与命令行执行效果不一致

![Maven_Installations1.png](/images/Maven/MavenUsageSummary/Eclipse/Maven_Installations1.png)

![Maven_Installation2.png](/images/Maven/MavenUsageSummary/Eclipse/Maven_Installation2.png)

![Maven_Installation3.png](/images/Maven/MavenUsageSummary/Eclipse/Maven_Installation3.png)

配置Maven的settings.xml文件

![Maven_Config.png](/images/Maven/MavenUsageSummary/Eclipse/Maven_Config.png)

修改settings.xml的本地仓库路径，添加一行配置

```xml
<localRepository>D:\SDK\Maven\m2_repository</localRepository>
```

配置setting.xml的中央仓库镜像为公司Maven私服，在\<mirrors/\>标签下添加

把下面的url替换成公司的Maven私服url

```xml
    <mirror>  
        <id>qf-public</id>  
        <mirrorOf>central</mirrorOf>  
        <url>http://org.jisonami:8081/nexus/content/groups/public/</url>  
    </mirror>
```

至此，Eclipse中关于Maven的配置就完成了

##### 项目创建

Eclipse创建项目底层使用的Maven插件还是maven-archetype-plugin，之前在命令怎么创建项目的，在Eclipse中还是一样的，只不过使用的是Eclipse的图形插件而已

###### 创建一个单模块jar项目

步骤截图如下

![NewMavenProject.png](/images/Maven/MavenUsageSummary/Eclipse/Simple/NewMavenProject.png)

![NewMavenProject2.png](/images/Maven/MavenUsageSummary/Eclipse/Simple/NewMavenProject2.png)

![NewMavenProject3.png](/images/Maven/MavenUsageSummary/Eclipse/Simple/NewMavenProject3.png)

![NewMavenProject4.png](/images/Maven/MavenUsageSummary/Eclipse/Simple/NewMavenProject4.png)

第一次创建项目时，Maven会下载相关依赖包，打开Window --> Show View --> Other --> General --> Progress，可以看到依赖下载的过程

![EclipseProgress.png](/images/Maven/MavenUsageSummary/Eclipse/EclipseProgress.png)

![DownloadDependency.png](/images/Maven/MavenUsageSummary/Eclipse/Simple/DownloadDependency.png)

依赖下载完成后，项目就创建完成了，目录结构如下

![DirectoryStructure.png](/images/Maven/MavenUsageSummary/Eclipse/Simple/DirectoryStructure.png)

###### 创建一个单模块war项目

步骤截图如下

![NewMavenProject.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/NewMavenProject.png)

![NewMavenProject2.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/NewMavenProject2.png)

![NewMavenProject3.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/NewMavenProject3.png)

![NewMavenProject4.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/NewMavenProject4.png)

第一次创建项目时，Maven会下载相关依赖包，打开Window --> Show View --> Other --> General --> Progress，可以看到依赖下载的过程

![EclipseProgress.png](/images/Maven/MavenUsageSummary/Eclipse/EclipseProgress.png)

![DownloadDependency.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/DownloadDependency.png)

依赖下载完成后，项目就创建完成了，目录结构如下

![ErrorDirectoryStructure.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/ErrorDirectoryStructure.png)

但是报错了，查看报错信息打开Window --> Show View --> Other --> General --> Markets/Problems

![Error.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/Error.png)

在pom.xml的\<dependencies/\>下加入以下依赖

```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.0.1</version>
        <scope>provided</scope>
    </dependency>
```

刷新Maven项目,对着项目单击右键

![EclipseUpdataMavenProject.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/EclipseUpdataMavenProject.png)

![EclipseUpdateMavenProject2.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/EclipseUpdateMavenProject2.png)

等待刷新完成后，目录结构如下

![DirectoryStructure.png](/images/Maven/MavenUsageSummary/Eclipse/WebApp/DirectoryStructure.png)

###### 创建一个多模块项目

操作步骤截图如下

创建一个app(parent)项目

![NewParentProject.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewParentProject.png)

![NewParentProject2.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewParentProject2.png)

创建app_common模块

![NewCommonModule.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewCommonModule.png)

![NewCommonModule2.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewCommonModule2.png)

![NewCommonModule3.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewCommonModule3.png)

![NewCommonModule4.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewCommonModule4.png)

![NewCommonModule5.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewCommonModule5.png)

创建app_web模块

![NewWebModule.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewWebModule.png)

![NewWebModule2.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewWebModule2.png)

![NewWebModule4.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewWebModule4.png)

![NewWebModule3.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewWebModule3.png)

![NewWebModule5.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/NewWebModule5.png)

app_web模块在Eclipse中报错，同样引入servlet-api依赖，处理方式参考单模块war项目

```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.0.1</version>
        <scope>provided</scope>
    </dependency>
```

最终目录结构如下

![DirectoryStructure.png](/images/Maven/MavenUsageSummary/Eclipse/MutiModule/DirectoryStructure.png)

##### 项目导入

操作步骤截图如下

![EclipseMavenImport.png](/images/Maven/MavenUsageSummary/Eclipse/Import/EclipseMavenImport.png)

![EclipseMavenImport.png2](/images/Maven/MavenUsageSummary/Eclipse/Import/EclipseMavenImport2.png)

![EclipseMavenImport.png3](/images/Maven/MavenUsageSummary/Eclipse/Import/EclipseMavenImport3.png)

![EclipseMavenImport.png4](/images/Maven/MavenUsageSummary/Eclipse/Import/EclipseMavenImport4.png)

##### 执行Maven构建命令

截图步骤如下

对着需要构建的项目单击右键，

![MavenBuild.png](/images/Maven/MavenUsageSummary/Eclipse/Build/MavenBuild.png)

我们可以看到Eclipse的Maven插件提供了好几种构建方式，有clean、test、install，还有输入任意的Maven任务进行执行，如下图所示，我们执行了clean package，与命令行运行Maven相比，少了mvn，其它是一样的，Eclipse执行Maven任务时会自动加上mvn前缀的

通过这种方式可以执行前面命令行介绍的所有Maven任务

![CleanPackage.png](/images/Maven/MavenUsageSummary/Eclipse/Build/CleanPackage.png)

然后发现报错了，

![Error.png](/images/Maven/MavenUsageSummary/Eclipse/Build/Error.png)

解决方式是设一个环境变量M2_HOME指向你的maven安装目录M2_HOME=D:\SDK\Maven\apache-maven-3.3.3

![M2_HOME.png](/images/Maven/MavenUsageSummary/M2_HOME.png)

然后在Window->Preference->Java->Installed JREs->Edit，在Default VM arguments中设置-Dmaven.multiModuleProjectDirectory=$M2_HOME

![Eclipse_JDK_Config.png](/images/Maven/MavenUsageSummary/Eclipse/Build/Eclipse_JDK_Config.png)

重复执行前面的操作

![BuildSucess.png](/images/Maven/MavenUsageSummary/Eclipse/Build/BuildSucess.png)

#### IntelliJ IDEA中使用Maven

##### 环境配置

单个项目配置Maven

![IdeaMavenSettings.png](/images/Maven/MavenUsageSummary/IDEA/IdeaMavenSettings.png)

![IdeaMavenSettings2.png](/images/Maven/MavenUsageSummary/IDEA/IdeaMavenSettings2.png)

全局Maven配置，所有项目公用

![GlobalMavenSettings.png](/images/Maven/MavenUsageSummary/IDEA/GlobalMavenSettings.png)

![GlobalMavenSettings2.png](/images/Maven/MavenUsageSummary/IDEA/GlobalMavenSettings2.png)

##### 项目创建

###### 创建一个单模块jar项目

操作步骤截图如下

![NewMavenProject.png](/images/Maven/MavenUsageSummary/IDEA/Simple/NewMavenProject.png)

![NewMavenProject2.png](/images/Maven/MavenUsageSummary/IDEA/Simple/NewMavenProject2.png)

![NewMavenProject3.png](/images/Maven/MavenUsageSummary/IDEA/Simple/NewMavenProject3.png)

![NewMavenProject4.png](/images/Maven/MavenUsageSummary/IDEA/Simple/NewMavenProject4.png)

![NewMavenProject4.1.png](/images/Maven/MavenUsageSummary/IDEA/Simple/NewMavenProject4.1.png)

![NewMavenProject4.2.png](/images/Maven/MavenUsageSummary/IDEA/Simple/NewMavenProject4.2.png)

![NewMavenProject5.png](/images/Maven/MavenUsageSummary/IDEA/Simple/NewMavenProject5.png)

目录结构如下

![DirectoryStructure.png](/images/Maven/MavenUsageSummary/IDEA/Simple/DirectoryStructure.png)

###### 创建一个单模块war项目

操作步骤截图如下

![NewMavenProject.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject.png)

![NewMavenProject1.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject1.png)

![NewMavenProject2.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject2.png)

![NewMavenProject3.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject3.png)

![NewMavenProject4.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject4.png)

![NewMavenProject4.1.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject4.1.png)

![NewMavenProject4.2.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject4.2.png)

![NewMavenProject5.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/NewMavenProject5.png)

目录结构如下

![DirectoryStructure.png](/images/Maven/MavenUsageSummary/IDEA/WebApp/DirectoryStructure.png)

webapp模块在导入Eclipse时会提示缺少servlet-api依赖，为了跨IDE，我们加入一下依赖

```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.0.1</version>
        <scope>provided</scope>
    </dependency>
```

###### 创建一个多模块项目

创建一个app(parent)项目

![NewMavenProject.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewMavenProject.png)

![NewMavenProject2.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewMavenProject2.png)

![NewMavenProject3.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewMavenProject3.png)

![NewMavenProject4.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewMavenProject4.png)

创建一个app_common模块

![NewCommonModule.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewCommonModule.png)

![NewCommonModule2.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewCommonModule2.png)

![NewCommonModule3.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewCommonModule3.png)

![NewCommonModule4.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewCommonModule4.png)

![NewCommonModule4.1.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewCommonModule4.1.png)

![NewCommonModule4.2.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewCommonModule4.2.png)

![NewCommonModule5.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewCommonModule5.png)

创建一个app_web模块

![NewWebModule.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewWebModule.png)

![NewWebModule2.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewWebModule2.png)

![NewWebModule3.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewWebModule3.png)

![NewWebModule4.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewWebModule4.png)

![NewWebModule4.1.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewWebModule4.1.png)

![NewWebModule4.2.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewWebModule4.2.png)

![NewWebModule5.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/NewWebModule5.png)

最终的目录结构如下

![DirectoryStructure.png](/images/Maven/MavenUsageSummary/IDEA/Mutimodule/DirectoryStructure.png)

##### 项目导入

操作步骤截图如下

![IdeaMavenImport.png](/images/Maven/MavenUsageSummary/IDEA/Import/IdeaMavenImport.png)

![IdeaMavenImport2.png](/images/Maven/MavenUsageSummary/IDEA/Import/IdeaMavenImport2.png)

目录结构如下

![DirectoryStructure.png](/images/Maven/MavenUsageSummary/IDEA/Import/DirectoryStructure.png)

##### 执行Maven构建命令

使用IDEA的Maven管理插件执行Maven构建命令

![IdeaMavenBuild.png](/images/Maven/MavenUsageSummary/IDEA/Build/IdeaMavenBuild.png)

![IdeaMavenBuild2.png](/images/Maven/MavenUsageSummary/IDEA/Build/IdeaMavenBuild2.png)

自定义运行Maven命令，执行的命令不需要mvn前缀

![CustomMavenRun.png](/images/Maven/MavenUsageSummary/IDEA/Build/CustomMavenRun.png)

![CustomMavenRun2.png](/images/Maven/MavenUsageSummary/IDEA/Build/CustomMavenRun2.png)

![CustomMavenRun2.1.png](/images/Maven/MavenUsageSummary/IDEA/Build/CustomMavenRun2.1.png)

![CustomMavenRun2.2.png](/images/Maven/MavenUsageSummary/IDEA/Build/CustomMavenRun2.2.png)

![CustomMavenRun3.png](/images/Maven/MavenUsageSummary/IDEA/Build/CustomMavenRun3.png)