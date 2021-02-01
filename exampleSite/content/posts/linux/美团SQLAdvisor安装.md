---
title: 美团SQLAdvisor安装
date: 2018-03-11 10:38:10
tags: ["环境搭建"]
categories: ["环境搭建"]
---
&emsp;&emsp;安装过程有点心塞，折腾了好久，主要问题是https://github.com/Meituan-Dianping/SQLAdvisor 的master分支的代码似乎有问题，导致安装到最后一步总是出现错误：
<!--more-->
```shell
/home/alertcode/SQLAdvisor/sqladvisor/main.cc:6:24: fatal error: sql/mysqld.h: No such file or directory
#include "sql/mysqld.h"
^
compilation terminated.
make[2]: *** [CMakeFiles/sqladvisor.dir/main.cc.o] Error 1
make[1]: *** [CMakeFiles/sqladvisor.dir/all] Error 2
make: *** [all] Error 2
```
换了一个包，https://github.com/Meituan-Dianping/SQLAdvisor/archive/v2.0.zip  ,<br/>重复 https://github.com/Meituan-Dianping/SQLAdvisor/blob/master/doc/QUICK_START.md 的安装步骤，在一片警告中，终于成功，内心是窃喜的。
参照官方文档,安装并运行成功后的效果如下：
```
2018-03-11 10:15:59 18369 [Note] 第1步: 对SQL解析优化之后得到的SQL:select `*` AS `*` from `application_config`.`spring_application` where (`application_name` like '%Train%')
2018-03-11 10:15:59 18369 [Note] 第2步：开始解析where中的条件:(`application_name` like '%Train%')
2018-03-11 10:15:59 18369 [Note] 第3步：条件(`application_name` like '%Train%') 是like前缀匹配,跳过不分析.
2018-03-11 10:15:59 18369 [Note] 第4步：表spring_application 的SQL太逆天,没有优化建议
2018-03-11 10:15:59 18369 [Note] 第5步: SQLAdvisor结束!
```
之后打算在这个上面套一个java web壳，膜拜C法，还是C比较屌气。。。
