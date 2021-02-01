---
title: shadowsocks+kvm+bbr梯子搭建
date: 2018-03-18 16:23:29
tags: ["环境搭建"]
categories: ["环境搭建"]
---

日常工作免不了要找个资料什么的，最近喜欢上了 YouTube，上面有些视频还是不错的，可以学个外语什么的，除此之外，google 搜索的质量也比较高。

<!--more-->

### 环境准备：

- 需要 kvm 服务器：[hostmybytes](http://www.hostmybytes.com/)，支持支付宝购买,12\$,大约 78 元左右
- shodowsocks 客户端：[shodowsocks-window](https://github.com/shadowsocks/shadowsocks-windows/releases)

### shadowsocks 服务端搭建

1. yum install python-pip
2. pip install shadowsocks
3. vi /etc/shadowsocks.json
4. 单用户配置

```json
{
  "server": "my_server_ip",
  "server_port": 8388,
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "password": "mypassword",
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false
}
```

5. 多用户配置

```json
{
  "server": "my_server_ip",
  "port_password": {
    "2333": "mima12345",
    "6666": "mima12345"
  },
  "timeout": 600,
  "method": "aes-256-cfb"
}
```

6. 启动和停止

```shell
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```

### shadowsocks 客户端配置

如下图所示：
![img](360截图172905079712892.png)

### showdowsocks 优化

优化的前提是机器必须是 KVM 虚拟，满足条件后，可以参考这个博客 https://teddysun.com/489.html ，添加 bbr 加速，反正加速效果明显，YouTube 1.5-2.5MB/s，超清无压力
