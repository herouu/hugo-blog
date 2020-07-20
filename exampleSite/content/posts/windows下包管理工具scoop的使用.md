---
title: "Windows下包管理工具scoop的使用"
date: 2020-01-30T16:02:25+08:00
draft: false

categories: ["环境搭建"]  
---
工作学习中常见的几类操作系统无非就是linux,windows,macos。因使用的需要，不可避免的要安装各种软件。因此操作系统为了满足安装软件的需要，提供了不同的安装方式。比如linux，macos使用命令行包管理工具安装。说到命令行包管理工具，不得不惊叹linux系统的强大，如centos有yum,ubunut有apt。相对而言，windows就比较鸡肋，常常使用图形化界面安装软件。难道说windows就没有类似于linux系统的命令行包管理工具了吗？并非如此，查找得知windows下有两款工具，提供了类似的功能。其一是Chocolatey，另一个是Scoop。
<!--more-->
## 为什么是scoop
chocolatey的使用是基于管理员权限运行的，需要的安装权限比较高，对环境变量侵染比较严重，默认安装位置是c盘，如果想改变路径，需要获得专业版授权，因此chocolatey更像是一个商业软件，这是其弊端。优点是安装一步到位，包括软件的桌面快捷方式，环境变量等等。相对而言，scoop比较轻量，安装软件需要权限较低，无需管理员权限，对环境变量侵染较低，可以改变scoop包的默认安装位置。美中不足的是无法直接在桌面创建快捷方式，需要自己手动创建，scoop的软件源没有Chocolatey的多，但好在可以通过自定义的方式高度定制，对于日常工作开发而言，自认为是足够了。不选择chocolatey的主要原因是chocolatey的软件包安装路径默认是c盘且更改收费，这个日积月累下去，c盘的容量堪忧。

## scoop的安装

在 PowerShell中执行，这里会将scoop默认安装到c盘，可以修改安装路径
```shell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser

# 设置安装路径
$env:SCOOP='D:\scoop'
[environment]::setEnvironmentVariable('SCOOP',$env:SCOOP,'User')

Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
# 或
iwr -useb get.scoop.sh | iex
```

## scoop的bucket源

```shell
scoop install git # scoop更新安装依赖git

scoop bucket add extras
scoop bucket add jetbrains
scoop bucket add tomato https://github.com/zhoujin7/tomato.git
scoop bucket add dorado https://github.com/h404bi/dorado.git
scoop bucket add nonportable
```
## 使用scoop搭建快速开发环境

```shell
scoop install fork
scoop install go
scoop install gradle
scoop install hugo
scoop install IntelliJ-IDEA-Ultimate
scoop install maven 
scoop install mobaxterm
scoop install nodejs-lts
scoop install notepadplusplus
scoop install OracleJDK8
scoop install vscode
scoop install v2rayn
scoop install fluent-terminal-np
scoop install ccleaner
scoop install geekuninstaller
scoop install picgo
scoop install Registryworkshop
```

## 参考资料
https://github.com/rasa/scoop-directory/blob/master/by-apps.md

## scoop加速设置
因为scoop依赖git,而其源大多放在github上，国内对github访问有限制，所以为了加速git，需要设置proxy,
编辑.gitconfig文件添加下面代码
```shell script
[credential]
	helper = manager
[user]
	name = bob
	email = 2269648132@qq.com
[http]
	proxy = 'socks5://127.0.0.1:10808'
[https]
	proxy = 'socks5://127.0.0.1:10808'
[http "https://github.com"]
	proxy = socks5://127.0.0.1:10808

```
或
```shell script
git config --global http.proxy 'socks5://127.0.0.1:10808'
git config --global https.proxy 'socks5://127.0.0.1:10808'

#只对github.com
git config --global http.https://github.com.proxy socks5://127.0.0.1:10808

#取消代理
git config --global --unset http.https://github.com.proxy

```
