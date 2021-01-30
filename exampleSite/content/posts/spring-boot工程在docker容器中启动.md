---
title: spring_boot工程在docker容器中启动
date: 2019-03-23 00:07:30
tags: ["springboot","docker"]
categories: ['docker']
---

自己搭建的一个简单的 springboot 模板工程是不支持 docker 容器启动的，况且 docker 在经历了几年的洗礼之后，相对来讲也比较成熟。不像之前资料比较少。不知道大家有没有经历过这样的场景，我在开发环境、测试环境、uat 环境打包部署都很正常，但是一到了发布生产的时候就会出各种奇葩的问题，问题的最大根源就是环境不一致问题。如何解决这个问题，就是镜像技术。让 N 台机器使用的是一套环境，包括软件及相应的版本。以前有使用 vmware 的，这个东西相当于 clone 了一台机器出来，但是很耗资源。相对而言 docker 就比较轻量，而且资源的利用率比较高。
在 java 工程中应该如何使用 docker 呢？

<!--more-->

### pom.xml 文件中添加打包插件

```xml
<plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.9</version>
            <configuration>
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
                <buildArgs>
                    <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    <plugins>
```

### 工程中添加 Dockerfile

注意`Dockerfile`为固定名字,配置如下，文件路径跟 pom.xml 为同一级。

```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

### 使用 docker 编译打包并启动工程

- centos7 安装 docker

```
yum install docker
```

- 打包编译
  工程文件路径下进行打包

```
mvn clean package dockerfile:build
```

- 编译成功后

执行`docker images`出现 springboot 的 image
![img](https://bj.bcebos.com/v1/alertcode-blog/spring_boot工程在docker容器中启动/springboot-docker.png)

- 运行 docker image

```
 docker run -p 8080:8080 -t springboot/train-boot-demo
```

### 可能遇到的问题

- dockerfile-maven-plugin 尽量使用新版的`dockerfile:build`命令
- 注意开启防火墙项目需要相关接口
- centos 下执行 docker 端口报错
  https://www.jianshu.com/p/fafe728b636b
