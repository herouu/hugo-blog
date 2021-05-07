---
title: "C++_web中使用conan"
subtitle: ""
date: 2021-05-06T19:52:17+08:00
lastmod: 2021-05-06T19:52:17+08:00
draft: true
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
引用网上一句话来总结c++,一次编写，到处编译，还编译不过；或者说c++是一门面向编译器编程的语言。
打算用c++来作为自己的第二门编程语言，自己对web编程也比较熟悉，索性用c++来写一个web应用 。
学习一门编程语言最快的方式就是在实践中摸索，自己使用了c++的web框架oatpp。
oatpp官网的crud demo没有用到依赖管理工具。依赖少的话也就无所谓了，但是假如依赖很多，依赖的版本管理还是很麻烦的。如果java web工程中没有使用maven又引用了很多第三方依赖，想想就觉着很刺激，类比c++也是一样的。
可是c++的依赖管理相对来说比较混乱，比较流行的有vspkg(微软的)、conan等等。比较后最终选择了conan,原因是灵活，跟maven一样可以自建私服，conan可以从conan-center下载二进制包，加速本地编译，若center没有相应的二进制包则下载依赖的源码进行本地编译，编译后的结果存储在本地。除非编译时指定参数，否则依赖的查找顺序是本地缓存、conan-center。
这里对oatpp-example-crud这个工程引入了conan包管理工具，改造后的源码地址在
[example-crud](https://github.com/herouu/example-crud.git)
<!--more--> 
## 在linux中使用
安装conan 
```shell
pip3 install conan
```
conan配置,compiler.libcxx=libstdc++为默认选项，修改为libstdc++11，否则会出现ABI问题
```shell
[settings]
os=Linux
os_build=Linux
arch=x86_64
arch_build=x86_64
compiler=gcc
compiler.version=9
compiler.libcxx=libstdc++11
build_type=Release
[options]
[build_requires]
[env]

```
```shell
mkdir build
cd build
conan install .. --build
cmake .. 
make
```
其他请参考
[C++ 包管理器 conan (wtih cmake) 简介](https://zhuanlan.zhihu.com/p/308733954)
## 在windows中使用
conan 默认配置
```shell
[settings]
os=Windows
os_build=Windows
arch=x86_64
arch_build=x86_64
build_type=Release
compiler=Visual Studio
compiler.version=16
compiler.runtime=MD
[options]
[build_requires]
[env]
```

```shell
mkdir winbuild
cd winbuild
conan install .. --build
cmake .. 
cmake --build . --config Release
```
