---
title: 使用Dockerfile构建镜像境搭建
date: 2018-03-30 01:59:34
tags: ["docker"]
categories: ['docker']
---

docker 创建镜像有两种方式

- 使用 commit 方式
- 使用 Dockerfile，推荐使用
  有以下优点：
  创建成功率高，且 commit 方式创建的镜像，运行的时候容易出问题

<!--more-->

### centos Dockerfile

```shell
    FROM centos
    MAINTAINER www.alertcode.top 2269648132@qq.com
    RUN yum install -y openssh-server
    RUN mkdir /var/run/sshd
    RUN ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N '' && \
    ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' && \
    ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key  -N ''
    RUN echo "root:123456" | chpasswd
    ENTRYPOINT ["/usr/sbin/sshd","-D"]
    EXPOSE 22
```

在 windows 系统中新建一个文件夹 centos-dockerfile
将上面的代码保存到名为 Dockerfile 的文件
运行：
docker build -t centos:ssh D://docker/centos-docker/
docker run -d -p 2222:22 centos:ssh
连接：
用 xshell 连接 2222 端口 账号：密码 => root:123456

### ubuntu Dockerfile

```shell
    FROM ubuntu
    MAINTAINER www.alertcode.top 2269648132@qq.com
    RUN apt-get update -y
    RUN apt-get install -y openssh-server
    RUN mkdir /var/run/sshd
    RUN echo "root:123456" | chpasswd
    #允许root用户以任何认证方式登陆（用户密码认证和公钥认证）
    RUN sed -i 's/prohibit-password/yes/g' /etc/ssh/sshd_config
    ENTRYPOINT ["/usr/sbin/sshd","-D"]
    EXPOSE 22
```

在 windows 系统中新建一个文件夹 ubuntu-dockerfile
将上面的代码保存到名为 Dockerfile 的文件
运行：
docker build -t ubuntu:ssh D://docker/ubuntu-docker/
docker run -d -p 2223:22 ubuntu:ssh
连接：
用 xshell 连接 2223 端口 账号：密码 => root:123456
