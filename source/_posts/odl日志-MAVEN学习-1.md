title: MAVEN学习-1
date: 2014-05-16 12:54:04
categories: 学习Maven
tags: [Maven]
---
参考文献：
1. [maven用途、核心概念、用法、常用参数和命令、扩展](http://trinea.iteye.com/blog/1290898)
2. [Maven官方文档](http://maven.apache.org/guides/index.html)
3. [Maven权威指南](http://www.sonatype.com/books/maven-book/reference_zh/public-book.html)
4. [maven安装](http://maven.apache.org/download.html)
5. [理解maven的核心概念](http://www.cnblogs.com/holbrook/archive/2012/12/24/2830519.html)

# MAVEN是什么
Maven 是为 Java™ 开发人员提供的一个极为优秀的**构建工具**，也可以使用它来管理您的项目生命周期。
作为一个生命周期管理工具，Maven 是基于阶段操作的，而不像 Ant 是基于 “任务” 构建的。

Maven 完成项目生命周期的所有阶段，包括验证、代码生成、编译、测试、打包、集成测试、安装、部署、以及项目网站创建和部署。

<!--more-->

## Pom
pom是指project object Model。

pom是一个xml，是maven工作的基础，在执行task或者goal时，maven会去项目根目录下读取pom.xml获得需要的配置信息。

pom文件中包含了项目的信息和maven build项目所需的配置信息，通常有项目信息(如版本、成员)、项目的依赖、插件和goal、build选项等等

pom是可以继承的，通常对于一个大型的项目或是多个module的情况，子模块的pom需要指定父模块的pom。

#####groupId:artifactId:version唯一确定了一个artifact

### Pom里面的元素
1. project pom文件的顶级元素
  
2. modelVersion 所使用的object model版本，为了确保稳定的使用，这个元素是强制性的。除非maven开发者升级模板，否则不需要修改  

3. groupId 是项目创建团体或组织的唯一标志符，通常是域名倒写，如groupId  5. org.apache.maven.plugins就是为所有maven插件预留的  

4. artifactId 是项目artifact唯一的基地址名  

5. packaging artifact打包的方式，如jar、war、ear等等。默认为jar。这个不仅表示项目最终产生何种后缀的文件，也表示build过程使用什么样的<a target="_blank" href="http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings">lifecycle</a>。  

5. version artifact的版本，通常能看见为类似0.0.1-SNAPSHOT，其中SNAPSHOT表示项目开发中，为开发版本  

6. name 表示项目的展现名，在maven生成的文档中使用
  
7. url表示项目的地址，在maven生成的文档中使用
  
8. description 表示项目的描述，在maven生成的文档中使用
  
9. dependencies 表示依赖，在子节点dependencies中添加具体依赖的groupId artifactId和version
  
10. build 表示build配置 
 
11. parent 表示父pom  


### Repositories
Repositories是用来存储Artifact的。如果说我们的项目产生的Artifact是一个个小工具，那么Repositories就是一个仓库，里面有我们自己创建的工具，也可以储存别人造的工具，我们在项目中需要使用某种工具时，在pom中声明dependency，编译代码时就会根据dependency去下载工具（Artifact），供自己使用。

同时，自己的项目完成后可以通过mvn install命令将项目放到仓库（Repositories）中仓库分为本地仓库和远程仓库，远程仓库是指远程服务器上用于存储Artifact的仓库，本地仓库是指本机存储Artifact的仓库，在系统.m2/repository下面。

### Denpendency
依赖关系的管理是maven最为人称道的地方。一个工程可以依赖多个其他工程， 通过工程的唯一标识（groupId+artifactId+version)可以明确指明依赖的库及版本，而且能够处理 依赖关系的传递。 maven可以指定依赖的作用范围（scope）

#### 更多依赖关系学习
[ 一起学Maven(Maven的依赖管理特性)](http://blog.csdn.net/songdeitao/article/details/18765405)

---
## SVM
这里说的版本管理（version management）不是指版本控制（version control）

版本管理中说得版本是指构件（artifact）的版本，而**非源码**的版本（如subversion中常见的rXXX，或者git中一次提交都有个sha1的commit号）

### SNAPSHOT

Maven有SNAPSHOT版本的概念，它与release版本对应，后者是指1.0，1.1，2.0这样稳定的发布版本。

假设现在乙可以将B的版本设置成1.0-SNAPSHOT，每次更改后，都`mvn deploy`到nexus中，每次deploy，maven都会将SNAPSHOT改成一个当前时间的timestamp，比如B-1.0-SNAPSHOT.jar到nexus中后，会成为这个样子：B-1.0-20081017-020325-13.jar。
Maven在处理A中对于B的SNAPSHOT依赖时，会根据这样的timestamp下载最新的jar，默认Maven每天 更新一次，如果你想让Maven**强制更新**，可以使用-U参数，如：`mvn clean install -U` 。

现在事情简化成了这个样子：乙做更改，然后mvn deploy，甲用最简单的maven命令就能得到最新的B。

---
## Archetype
maven创建项目是根据Archetype（原型）创建

Archetype对于项目的作用就相当于模具对于工具的作用，我们想做一个锤子，将铁水倒入模具成型后，稍加修改就可以了。

我们可以根据项目类型的需要使用不同的Archetype创建项目。通过Archetype我们可以快速标准的创建项目。利用Archetype创建完项目后都有标准的文件夹目录结构。

### quick start工程

创建一个简单的quick start项目，指定 -DarchetypeArtifactId为maven-archetype-quickstart，如下命令
```xml
mvn archetype:generate \
-DgroupId=org.amson.openflow \
-DartifactId=testing \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DinteractiveMode=false
```  
其中DgroupId指定groupId，DartifactId指定artifactId，DarchetypeArtifactId指定ArchetypeId，DinteractiveMode表示是否使用交互模式，交互模式会让用户填写版本信息之类的，非交互模式采用默认值。这样我们便建好了一个简单的maven项目。

（也可以利用eclipse简单方便快捷创建maven项目）
