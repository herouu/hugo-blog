---
title: 内网映射frp服务器环境搭建
date: 2018-03-21 20:59:34
tags: ["环境搭建"]
categories: ["环境搭建"]
---

&emsp;&emsp;前不久折腾了一个梯子，梯子的内存比较小，然后提供的服务有限，我只是用了一下梯子的流量而已。假如自己有一台旧的台式机，怎样把台式机当成服务器，从内网向外网提供服务呢？这就不得不提到内网穿透，这个东西。之前，内网穿透是用的花生壳，但是花生壳越来越贵，自然而然也就用不起了，但是，花生壳的原理，跟下面说道的这个内网穿透工具如出一辙。
&emsp;&emsp;基本上是要有一个公网的 Ip,然后在拥有公网 Ip 的主机部署 frp 服务端，然后服务端通过 ssh 协议访问局域网内的客户端，客户端机器部署自己向公网提供的服务，拥有公网 Ip 的那台主机相当于一个跳板。把 TCP 连接到局域网内。这样的好处是什么？就是省钱。像国内的云主机，价格偏贵，且提供的服务质量渣的要死，无力吐槽。国外的服务，又不是很稳定，且基于 KVM 虚拟的服务器价格也很高，像 openVZ 虚拟的机器，访问速度有些稍慢。目前来讲最好的搭建方式就是国外 KVM+bbr 加速，访问速度可以，性价比比较高。偶尔，看个 YouTube，高清无压力，快哉快哉。这里说一下，frp 这个东西，自用就好，不适合上生产。评论说 ngrok 这个东西可以，搭建教程放在这里：[内网穿透(NAT 穿透)之 ngrok 搭建服务器](https://blog.csdn.net/hpf247/article/details/55830106)
&emsp;&emsp;下面说一下 frp 映射的搭建（这个东西，用 go 语言，国人良心之作，估计要被约谈）。项目托管在 github 上，地址：[fatedier / frp](https://github.com/fatedier/frp)
具体搭建参照[[项目中文文档]](https://github.com/fatedier/frp/blob/master/README_zh.md)

<!--more-->

遇到了几个坑这里说一下

1. 遇到下面这个错误，以为是公钥，秘钥的问题折腾很久

```shell
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:niFEzRGEZ0EE60D4Ha9qaaoeeYOOJ2Ur3kp5m9tQGmk.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending RSA key in /root/.ssh/known_hosts:1
RSA host key for [XXX.XXX.XXX.XXX]:6000 has changed and you have requested strict checking.
```

**解决办法：**
运行下面这个命令`rm -rf ~/.ssh/known_hosts`重新连接一下就好了

2. 第二个坑是：执行`ssh -oPort=6000 test@x.x.x.x`成功，

- client 配置如下：

```
  #frpc.ini
  [common]
  server_addr = XXX.XXX.156.213
  server_port = 7000
  [ssh]
  type = tcp
  local_ip = 127.0.0.1
  local_port = 22
  remote_port = 6000
  [web]
  type = http
  #本地启动的服务
  local_port = 8080
  # 配置自己的域名，一定不要跟server_addr相同
  custom_domains = XXX.alertcode.top
```

- server 配置如下：

```
[common]
bind_port = 7000
vhost_http_port = 80
[ssh]
#type = tcp
#local_ip = 127.0.0.1
#local_port = 22
#remote_port = 6000
[common]
dashboard_port = 7500
# dashboard's username and password are both optional，if not set, default is admin.
dashboard_user = admin
dashboard_pwd = admin
```

然后，就死活解析不到本地，蛋疼。直到看到文档里的这句话（ www.yourdomain.com 的域名 A 记录解析到 IP x.x.x.x，如果服务器已经有对应的域名，也可以将 CNAME 记录解析到服务器原先的域名）的意思是域名要不一致才能映射。
总之，折腾成功。
&emsp;&emsp;这一周，把以前的旧 Android 手机,刷成 linux 系统，然后自建验证服务器什么的，像 idea,jrebel 这样的验证。然后试一下，能不能用手机构建一个 maven 私有库。资金充裕的话，还是要购买正版。以后，应用部署的话，直接用自己的旧机器就好了，KVM 512MB 内存，1 核别看这么鸡肋的配置，能干的事情多着能，linux 是个好东西。
