---
title: 如何配置openwrt的透明代理
date: 2019-03-02 16:07:40
tags: ["环境搭建"]

---

&emsp;&emsp;自己买了 GL.inet 家的路由器(基于 openwrt),因为 openwrt 的固件可玩性比较高，自己就折腾了一下，折腾的目的就是配置透明代理,相应的就了解了一下透明代理、DNS 解析以及转发
&emsp;&emsp;这里声明一下：主要就是为了查找资料并学习

<!--more-->

&emsp;&emsp;下面介绍一下自己踩坑的过程

## 安装相应的包

&emsp;&emsp;主要有以下几个包，功能介绍如下：

```shell
# ChinaDNS 服务
opkg install ChinaDNS
# ChinaDNS 界面
opkg install luci-app-chinadns
#  dns转发
opkg install dns-forwarder
#  dns转发 界面
opkg install luci-app-dns-forwarder
#  ss
opkg install shadowsocks-libev
# ss界面
opkg install luci-app-shadowsocks
```

&emsp;&emsp;推荐通过安装脚本安装

```shell
wget http://openwrt-dist.sourceforge.net/auto_install.sh
chmod +x auto_install.sh
./auto_install.sh
```

&emsp;&emsp;这个安装脚本有点问题 ，会提示 shadowsocks-libev-static 包找不到，替换成 shadowsocks-libev,安装

```
 opkg install shadowsocks-libev
 opkg install iptables-mod-tproxy
```

&emsp;&emsp;更新 ChinaDNS 路由表

```shell
wget -O /tmp/delegated-apnic-latest 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' && awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' /tmp/delegated-apnic-latest > /etc/chinadns_chnroute.txt
```

## 配置

### 服务->影梭设置

#### 服务器管理 添加服务器

![ss_服务器管理.png](images/如何配置openwrt的透明代理/ss_服务器管理.png "如何配置openwrt的透明代理")

#### 访问控制

![ss_访问控制.png](images/如何配置openwrt的透明代理/ss_访问控制.png "如何配置openwrt的透明代理")

#### 基本设置 开启透明代理

![ss_基本设置.png](images/如何配置openwrt的透明代理/ss_基本设置.png "如何配置openwrt的透明代理")

### DNS 转发设置

![DNS转发设置.png](images/如何配置openwrt的透明代理/DNS转发设置.png "如何配置openwrt的透明代理")

### chinaDNS 设置

![chinaDNS设置.png](images/如何配置openwrt的透明代理/chinaDNS设置.png "如何配置openwrt的透明代理")

### 网络-> DHCP/DNS 设置

#### 基本设置

![DHCP_DNS基本设置.png](images/如何配置openwrt的透明代理/DHCP_DNS基本设置.png "如何配置openwrt的透明代理")

#### hosts 和解析文件设置

![DHCP_DNS_hosts和解析文件.png](images/如何配置openwrt的透明代理/DHCP_DNS_hosts和解析文件.png "如何配置openwrt的透明代理")
&emsp;&emsp;这些设置完毕，透明代理也就完成了，查个资料还是很方便的。

## 参考资料：

[在 OpenWRT/LEDE 上安装配置 Shadowsocks 服务](https://wody11.blogspot.com/2018/10/openwrtlede-shadowsocks.html)
[OpenWRT 路由器上的 Shadowsocks 的安装与配置](https://blog.wqlin.com/openwrt-ss.html)
[零基础自制 openwrt 智能分流路由器](https://velaciela.ms/how-to-use-openwrt-fuckgfw)
