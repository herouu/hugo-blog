---
title: maven私服nexus环境搭建搭建
date: 2018-03-12 22:28:45
tags: ["环境搭建"]
categories: ["环境搭建"]
---

&emsp;&emsp;初衷：因为要折腾代码，所以像maven私服这个东西还是要有的，自己方便
<!--more-->
** 1.  maven私服搭建的过程主要参照**
https://www.cnblogs.com/aegisada/p/6323067.html
进行环境搭建。
** 2.  maven 环境搭建，然后进行jar包上传**
*  maven setting.xml 配置

```xml
    <servers>
        <server>
         <!--releases 连接发布版本项目仓库-->
          <id>releases</id>
          <!--访问releases这个私服上的仓库所用的账户和密码-->
          <username>***</username>
          <password>***</password>
        </server>
        <server>
        <!--snapshots 连接测试版本项目仓库-->
          <id>snapshots</id>
          <!--访问releases这个私服上的仓库所用的账户和密码-->
          <username>***</username>
          <password>***</password>
        </server>
    <servers>
```

* 工程的pom.xml文件中增加如下配置，为了能连接到maven私服

```xml
     <distributionManagement>
        <!--pom.xml 这里<id> 和 settings.xml 配置 <id> 对应 id 为nexus的库名  -->
        <repository>
          <id>releases</id>
          <url>http://****/nexus/content/repositories/releases/</url>
        </repository>
        <snapshotRepository>
          <id>snapshots</id>
          <url>http://****/nexus/content/repositories/snapshots/</url>
        </snapshotRepository>
      </distributionManagement>
```
&emsp;&emsp;根据工程的版本号决定上传到哪个宿主仓库，如果版本为release则上传到私服的release仓库，如果版本为snapshot则上传到私服的snapshot仓库。

**  3.   maven私服的使用 **
      &emsp;&emsp;之前一直使用的阿里云的 maven 库，直接把阿里云的maven 仓库变成自己的代理仓库，
      maven仓库有几个等级，本地仓库->宿主仓库->代理仓库->中心仓库（也是maven依赖的查找顺序）
      自己的私服相当于宿主仓库，阿里云为代理仓库，中心仓库就是(https://repo1.maven.org/maven2/) 这个仓库(大概是这个，官方仓库)。
      跟引用aliyun的仓库一致

```xml
<mirrors>
         <mirror>
            <id>nexus-aliyun</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus aliyun</name>
            <url>http://****/nexus/content/groups/public</url>
        </mirror>
        <!--  <mirror>  这里aliyun maven仓库因为私服设置成了proxy，所以在配置中不再引用
            <id>nexus-aliyun</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus aliyun</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
        </mirror> -->
  </mirrors>
```

** 4.  主要参考了两篇文章：这两篇还是比较靠谱的 **
  *   [nexus-2.14.2-01-bundle构建maven私服](https://www.cnblogs.com/aegisada/p/6323067.html)
  *   [nexus搭建maven私服及私服jar包上传和下载 ](http://blog.csdn.net/nocol123/article/details/73838089)
**5.  更高级的玩法**
[Jenkins+Maven+SVN（或Git）+Nexus 搭建持续集成环境](https://www.cnblogs.com/xiaoyaojinzhazhadehangcheng/articles/8302491.html)
目前公司再用，个人折腾，有个maven 私服足矣。
** 6. 上传archetype 模板工程到个人私服 **
主要有三步：
* 在工程目录下执行mvn archetype:create-from-project
* 在生成的target\generated-sources\archetype下执行mvn install,这一步是在本地的maven库中生成模板工程
* 在生成的target\generated-sources\archetype下执行mvn deploy,这一步是将模板工程部署到远程私人maven库
注意：都需要在相应的pom.xml文件里添加配置
```xml
  <distributionManagement>
    <repository>
      <id>releases</id>
      <url>http://****/nexus/content/repositories/releases/</url>
    </repository>
    <snapshotRepository>
      <id>snapshots</id>
      <url>http://****/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
  </distributionManagement>
```
