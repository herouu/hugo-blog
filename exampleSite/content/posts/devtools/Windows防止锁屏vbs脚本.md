---
title: "Windows防止锁屏vbs脚本"
subtitle: ""
date: 2021-12-13T16:58:56+08:00
lastmod: 2021-12-13T16:58:56+08:00
draft: false
author: ""
authorLink: ""
description: ""

tags: []
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---
办公环境使用虚拟机，通过终端机连接，虚拟机超过一定时间自动锁屏，为防止锁屏，记录使用的脚本
<!--more-->

## never-lockout.vbs脚本
```vbscript
Set wshShell = WScript.CreateObject("Wscript.shell") 
Dim i
i=0
do
i=i+1
WScript.Echo i
WScript.Sleep 10000
'或使用NUMLOCK'
wshShell.SendKeys "{CAPSLOCK}"
WScript.Sleep 500
wshShell.SendKeys "{CAPSLOCK}"
Loop
```

## never.bat
```bat
echo off
cscript never-lockout.vbs
pause
```