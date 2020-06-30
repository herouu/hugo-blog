---
title: dokku环境搭建
date: 2019-04-25 22:24:54
tags: ["环境搭建"]

---

### 安装 dokku

```shell
wget https://raw.githubusercontent.com/dokku/dokku/v0.15.5/bootstrap.sh;
sudo DOKKU_TAG=v0.15.5 bash bootstrap.sh
```

<!--more-->

### 配置 ssh

&emsp;&emsp;采用第二种方式，将本地的 id_rsa.pub 上传到 dokku 服务器任意路径下执行
`cat id_rsa.pub | sudo sshcommand acl-add dokku custom-identifier`

- 官方参考如下：

```
# assuming you have ssh access via root
cat ~/.ssh/id_rsa.pub | ssh root@dokku.com "sudo sshcommand acl-add dokku custom-identifier"
# assuming you are logged in as a user with root
cat path/to/id_rsa.pub | sudo sshcommand acl-add dokku custom-identifier
```

### 发布一个 app

1. 本地克隆 heroku demo 代码

```
#local
git clone git@github.com:heroku/ruby-getting-started.git
```

2. dokku 服务器端

```shell
dokku apps:create ruby-getting-started
sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git
dokku postgres:create railsdatabase
dokku postgres:link railsdatabase ruby-getting-started
```

3. 本地推送

```shell
#local
cd ruby-getting-started
git remote add dokku dokku@dokku.top:ruby-getting-started
git push dokku master
```

### 参考资料

- 遇到`fatal: Could not read from remote repository`问题 https://github.com/dokku/dokku/issues/1813
- 如何发布一个 app http://dokku.viewdocs.io/dokku/deployment/application-deployment/
