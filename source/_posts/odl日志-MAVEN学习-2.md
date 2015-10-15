title: MAVEN学习-2
date: 2014-05-16 14:54:04
categories: Maven
tags: [程序员初级修炼,学习：Maven]
---
参考文献：
1. [maven用途、核心概念、用法、常用参数和命令、扩展](http://trinea.iteye.com/blog/1290898)

主要介绍maven常用参数和命令以及简单故障排除


## mvn常用参数
mvn -e 显示详细错误
mvn -U 强制更新snapshot类型的插件或依赖库（否则maven一天只会更新一次snapshot依赖）
**mvn -o 运行offline模式，不联网更新依赖**

<!--more--> 

## Build Lifecycle中介绍的命令
mvn test-compile 编译测试代码
mvn test 运行程序中的单元测试
mvn  compile 编译项目
mvn package 打包，此时target目录下会出现maven-quickstart-1.0-SNAPSHOT.jar文件，即为打包后文件
mvn install 打包并安装到本地仓库，此时本机仓库会新增maven-quickstart-1.0-SNAPSHOT.jar文件。
每个phase都可以作为goal，如之前介绍的mvn clean install
 
 
## maven 常用命令
mvn archetype:generate 创建maven项目
mvn package 打包，上面已经介绍过了
mvn install 打包并安装到本地库
mvn site 生成项目相关信息的网站
 

## maven简单故障排除
 
mvn -Dsurefire.useFile=false，如果执行单元测试出错，用该命令可以在console输出失败的单元测试及相关信息
mvn -X， maven log level设定为debug在运行
mvn --help 