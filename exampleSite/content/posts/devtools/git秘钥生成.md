---
title: git秘钥生成
date: 2018-02-02 23:43:31
categories: ["环境搭建"]
---
&emsp;&emsp;Git是分布式的代码管理工具，远程的代码管理是基于SSH的，所以要使用远程的Git则需要SSH的配置。
<!--more-->

**github的SSH配置如下：**

### 一 、设置Git的user name和email：
```
$ git config --global user.name "user"
$ git config --global user.email "user@gmail.com"
```
### 二、生成SSH密钥过程：
#### 1.查看是否已经有了ssh密钥：
&emsp;&emsp;cd ~/.ssh 如果没有密钥则不会有此文件夹，有则备份删除
#### 2.生成密钥：
```
$ ssh-keygen -t rsa -C "user@gmail.com"
```
&emsp;&emsp;按3个回车，密码为空。
```
Your identification has been saved in /home/tekkub/.ssh/id_rsa.
Your public key has been saved in /home/tekkub/.ssh/id_rsa.pub.
The key fingerprint is:
```
&emsp;&emsp;最后得到了两个文件：id_rsa和id_rsa.pub
#### 3.添加密钥到ssh：
&emsp;&emsp;ssh-add 文件名,需要之前输入密码。
#### 4.在github上添加ssh密钥，
&emsp;&emsp;这要添加的是“id_rsa.pub”里面的公钥，打开https://github.com/ ，登陆github，然后添加ssh。

#### 5.测试：ssh git@github.com
```
The authenticity of host ‘github.com (207.97.227.239)’ can’t be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added ‘github.com,207.97.227.239′ (RSA) to the list of known hosts.
ERROR: Hi tekkub! You’ve successfully authenticated, but GitHub does not provide shell access
Connection to github.com closed.
```


### 三、 开始使用github
#### 1.获取源码：
```
$ git clone git@github.com:username/项目名.git
```
#### 2.git和svn不同
&emsp;&emsp;git是分布式的，没有服务器概念。所有的人的机器上都有一个repo，每次提交都是给自己机器的repo。

**仓库初始化：**
```
  git init
```
**生成快照并存入项目索引：**

```
  git add
```
**文件,还有git rm,git mv等等…项目索引提交：**

```
  git commit
```
#### 4.协作编程：
**将本地repo于远程的origin的repo合并，
推送本地更新到远程：**

```
git push origin master
```
**更新远程更新到本地：**

```
git pull origin master
```
**添加远端repo：**

```
$ git remote add upstream git://github.com/user/user-model.git
```
**重命名远端repo：**

```
$ git://github.com/user/user-model.git为“upstream”
```
