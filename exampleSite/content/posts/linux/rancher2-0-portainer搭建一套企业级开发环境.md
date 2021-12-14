---
title: rancher2.0+portainer快速搭建一套企业开发环境
date: 2019-04-28 11:37:46
tags: ["环境搭建","docker"]
categories: ['docker']

---

这里说一下为什么用两套差不多的都是容器管理工具，搭建一套环境。
因为主要用到他们的应用商店，两者收录的应用并不相同，避免自己去找 docker 镜像并验证的过程（避坑）。portainer 对 swarm 支持相对友好，而从 rancher2.0 开始， rancher2.0 的构建就是基于 k8s 的。恰好自己打算从 k8s 和 swarm 中选择 k8s 来学习，然后就有了这套环境的搭建过程。

<!--more-->

### 机器分布

- 部署 portainer
  192.168.1.113 centos-rancher
  centos-rancher 搭建单机 docker 服务 如：maven 私有仓库 nexus3、构建工具 jenkins、rabbitmq
- 部署 rancher2
  centos1、centos2、centos3 用于 k8s 集群， 主要用来底层组件集群环境的搭建如 zookeeper、kafka 等
  192.168.1.109 centos1
  192.168.1.110 centos2
  192.168.1.111 centos3

> 选用这两个容器管理工具，是因为他们有一个 app 商店这样的一个概念，部署一个 jenkins 服务方便到鼠标一点。对于从头到尾搭建一套适用于企业级的开发环境，或许也就是小半天，这对于从头到尾自己下包，改配置来说的传统部署方式是节省了不少时间。

### portainer 环境搭建

```
docker volume create portainer_data
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```

### 效果图

![portainer效果图](images/rancher2.0+portainer快速搭建一套企业开发环境/portainer运行效果图.png "portainer运行效果图")

### rancher2.0 +k8s 环境搭建

#### 在 centos-rancher 启动 rancher server

```
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

#### 开放 centos1、centos2、centos3 3 台机器的端口

参考https://rancher.com/docs/rancher/v2.x/en/installation/references/
开启防火墙端口，

参考命令如下

```
 firewall-cmd --zone=public --add-port=2376/tcp --permanent
 sudo firewall-cmd --reload

```

#### rancher2.0 配置
rancher2.0 的配置就是 在页面上点一点，唯一需要注意的是开启相应的端口
#### 注意

- centos1、centos2、centos3 3 台机器的 hostname 一定不要相同，自己明明在 3 台机器上都执行了 rancher-agent 镜像，但是总是提示自己只有一台机器被识别。
  以为自己是用的 kvm 镜像复制，结果复制的 hostname 相同，rancher 集群环境下识别为一台机器。
- 一定要根据 rancher 官方文档说的开放的端口进行防火墙配置。

> 临时修改 hostname 命令

> ```
> hostname 机器名
> ```

> 永久修改

> ```
> vi /etc/hostname
> ```

#### 应用商店

对于二者的应用商店来说，rancher2.0 比较适合构建大型分布式服务，比如 kafka 集群，而 portainer,比较适合单节点服务的部署，容器部署的成功率很高，对于搭建个人开发环境来说，也就是分分钟的事情。不论使用什么，相对于传统的安装软件的方式来说，还是可以节省不少时间的。
另外，rancher 相对来说比较臃肿，而 portainer 比较轻量，相对来说上手比较快，而 rancher2.0 基于 k8s，学习成本比较大。

