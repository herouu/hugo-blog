---
title: springboot使用Jenkins部署 
date: 2021-02-04 00:07:30 
tags: ["springboot","docker","jenkins"]
categories: ["springboot","docker","jenkins"]
---
使用jenkins发布一个springboot工程有如下几个基本步骤： clone代码->编译打包->ssh上传->启动部署脚本->验证是否发布成功
<!--more-->
* springboot工程jenkins普通发布
* springboot工程docker镜像jenkins普通发布
* 使用第三方持续集成工具,比如阿里云效

## springboot工程jenkins发布
### jenkins 安装
jenkins安装网上的教程很多，主要参考
[springboot(十六)：使用Jenkins部署Spring Boot](http://www.ityouknow.com/springboot/2017/11/11/spring-boot-jenkins.html )
注意：

* ~~jenkins容器的选择，自带的jetty容器的性能不太好，然后果断选择通过war包部署到tomcat的方式~~
* 使用docker容器安装jenkins
* 这三个插件Git plugin和Maven Integration plugin，publish over SSH应该是必须的
* 当代码管理使用的是github或gitlab时，要将生成的公钥放进git的setting->SSH Keys


## springboot工程docker镜像jenkins发布

#### springboot工程配置Dockerfile

* pom.xml 文件中添加打包插件

```xml

<properties>
  <docker.image.prefix>springboot</docker.image.prefix>
</properties>
<plugins>
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.13</version>
    <executions>
        <!-- 配置goal build使maven 在build阶段执行 配置该项 
        运行mvn clean package 即可以打包，否则需要运行 mvn clean package dockerfile:build -->
        <execution>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <repository>${docker.image.prefix}/${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
<plugins>
```
* 工程中添加 Dockerfile

注意`Dockerfile`为固定名字,配置如下，文件路径跟 pom.xml 为同一级。maven父子工程，需要将Dockerfile放置到boot子工程pom.xml同一级

```
FROM adoptopenjdk:8-jre-hotspot
MAINTAINER herouu 2269648132@qq.com
RUN mkdir /opt/app
ARG JAR_FILE
COPY ${JAR_FILE} /opt/app/ruoyi-admin.jar
ENTRYPOINT ["java","-jar","/opt/app/ruoyi-admin.jar"]
EXPOSE 8080
```

#### 使用宿主机docker编译打包并启动工程

- 打包编译 工程文件路径下进行打包
```shell
# pom.xml中的docker插件配置goal build使maven在build阶段自动执行docker插件，运行mvn clean package 即可以构建docker镜像，
# 否则需要运行 mvn clean package dockerfile:build 构建镜像
mvn clean package
```
ssh 远程连接宿主机运行 docker image
```shell
docker kill ruoyi-admin
docker rm ruoyi-admin 
docker run -d -p 8080:8080 --name ruoyi-admin -v /root/logs:/logs springboot/ruoyi-admin:3.3.0
```
![img](images/spring_boot工程在docker容器中启动/springboot-docker.png "spring_boot工程在docker容器中启动")


### 使用阿里云效
只要流水线配置好，从提交代码的那一刻开始，触发hook,流水线会执行clone、编译、打包、发布，只要不出错，基本上不用动手。jenkins也可以实现类似的功能，这需要部署jenkins服务，Jenkins构建项目比较耗性能。如果是公司开发，建议部署Jenkins，若是个人开发，小团队，建议使用第三方持续集成工具，毕竟每月有免费时长，羊毛不薅白不薅。本站的【[练手admin](http://101.200.79.90/)】，使用阿里云效进行持续集成。代码从提交到codeup，到同步github,同时构建到部署。让开发人员专注于撸码。这对个人开发用户来说确实省去了不少时间。

## 可能遇到的问题

- dockerfile-maven-plugin 尽量使用新版的`dockerfile:build`命令
- 注意开启防火墙项目需要相关接口
- centos 下执行 docker 端口报错
  https://www.jianshu.com/p/fafe728b636b
