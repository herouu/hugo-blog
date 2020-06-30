---
title: minio对象存储搭建
date: 2019-03-23 00:29:14
tags: ["环境搭建"]

---

&emsp;&emsp;开心到爆炸，接到短信，告诉我 ICP 备案取消，呵呵啊，心中顿时涌起了万千草泥马。
&emsp;&emsp;备案取消导致的直接结果就是托管在七牛云上的对象存储用不了，当然大部分就是博客用到的一些图片链接，然后就是之前已经上传的图片还不知道能不能找回来，真刺激。
&emsp;&emsp;因为云存储的不确定性，那么有没有方案替代国内的对象存储呢？(自然是避免 ICP 审核以及自己可控),github 上搜索 object storage ,找到了 minio
&emsp;&emsp;下面即是 minio 的搭建及使用教程，主要参考的文档[minio 中文文档](https://docs.minio.io/cn/)

<!--more-->

### 安装稳定版

```
docker pull minio/minio
```

```
docker run -p 9000:9000 --name minio1 \
 -e "MINIO_ACCESS_KEY=xxxxxxx" \
 -e "MINIO_SECRET_KEY=xxxxxxx" \
 -v /mnt/data:/data \
 -v /mnt/config:/root/.minio \
 minio/minio server /data
```

安装后使用浏览器访问 http://domain:9000，如果可以访问，则表示minio已经安装成功

### 解决 minio 永久链接问题

- 下载 mc
  wget https://dl.minio.io/client/mc/release/linux-amd64/mc
  chmod +x mc
  ln -s /root/mc /usr/bin

* 设置存储空间为 public
  mc config host add minio http://domain:9000 <YOUR-ACCESS-KEY> <YOUR-SECRET-KEY> S3v4
  mc policy public minio/alertcode
* 查看永久链接
  mc policy minio/alertcode

### 使用 TLS 安全的访问 Minio 服务

```
docker run -p 443:443 --name minio1   -e "MINIO_ACCESS_KEY=XXXX"   -e "MINIO_SECRET_KEY=XXXX"   -v /mnt/data:/data   -v /mnt/config:/root/.minio   minio/minio server --address ":443" /data
```

&emsp;&emsp;从此使用上了私有化对象存储
