title: MAVEN学习-3
date: 2014-05-16 16:54:04
categories: 学习Maven
tags: [Maven]
---
# ODL中Maven管理
1. 关闭MAVEN的测试功能

```
mvn clean install -DskipTests 
/* instead of "mvn clean install" */
```

2. 关闭MAVEN在线更新依赖库与插件

```
-o，--offline 离线模式工作
```

该参数可以阻止通过网络更新插件或依赖。

3. eclipse下关闭MAVEN的更新
![eclipse下关闭更新](http://askingwindy-gitcafe.qiniudn.com/5-16.png)

<!--more-->

---

# OSGI+MAVEN

#### how can maven-bundle-plugin find packages in Import-Package - Stack Overflow?
 a package is referenced in bytecode, the bundle plugin already knows its name and can include it in the import package statement, so it doesn't need to be "found". However, almost always anything referenced in the bytecode will be included as a maven dependency
[more](http://stackoverflow.com/questions/12445933/how-can-maven-bundle-plugin-find-packages-in-import-package)

## ODL里OSGI模块pom.xml文件修改

参考blog:[http://fredhsu.wordpress.com/2013/05/14/odl-maven-osgi/](http://app.yinxiang.com/shard/s8/sh/98dd2fa4-254a-4f21-895a-9b7cac40d03d/c47f8d0741c1395cfb59c4e3a01aa696)

参考代码：[odl-apps](https://github.com/askingwindy/odl-apps/blob/master/mystats/pom.xml)


1. line4-9：继承ODL项目中的父pom
2. line15：将packaging从jar改为bundle
3. line18-48：为OSGI声明flexi plugin

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.opendaylight.controller</groupId>
        <artifactId>commons.opendaylight</artifactId>
        <version>1.4.0-SNAPSHOT</version>
        <relativePath>../../../controller/opendaylight/commons/opendaylight</relativePath>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>mystats</artifactId>
    <name>mystats</name>
    <url>http://maven.apache.org</url>
    <packaging>bundle</packaging>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.felix</groupId>
                <artifactId>maven-bundle-plugin</artifactId>
                <version>2.3.6</version>
                <extensions>true</extensions>
                <configuration>
                    <instructions>
                        <Import-Package>
                            org.opendaylight.controller.sal.core,
                            org.opendaylight.controller.statisticsmanager,
                            org.opendaylight.controller.switchmanager,
                            org.opendaylight.controller.sal.utils,
                            org.opendaylight.controller.sal.reader,
                            org.opendaylight.controller.sal.flowprogrammer,
                            org.opendaylight.controller.sal.match,
                            org.slf4j,
                            org.apache.felix.dm
                        </Import-Package>
                        <Bundle-Activator>
                            com.example.mystats.Activator
                        </Bundle-Activator>
                        <Export-Package>
                            com.example.mystats
                        </Export-Package>
                    </instructions>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.opendaylight.controller</groupId>
            <artifactId>sal</artifactId>
            <version>0.4.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.opendaylight.controller</groupId>
            <artifactId>statisticsmanager</artifactId>
            <version>0.4.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```


