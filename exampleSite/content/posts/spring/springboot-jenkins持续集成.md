---
title: springboot+jenkins持续集成
date: 2018-07-09 20:59:51
tags: ["环境搭建"]
categories: ["环境搭建"]
---
&emsp;&emsp;后期上Docker,先满足需求，再优化
<!--more-->
### jenkins 安装
jenkins安装网上的教程很多，主要参考![springboot(十六)：使用Jenkins部署Spring Boot](http://www.ityouknow.com/springboot/2017/11/11/springboot-jenkins.html )
注意：
* jenkins容器的选择，自带的jetty容器的性能不太好，然后果断选择通过war包部署到tomcat的方式
* 这三个插件Git plugin和Maven Integration plugin，publish over SSH应该是必须的
* 当代码管理使用的是github或gitlab时，要将生成的公钥放进git的setting->SSH Keys

### 自动部署脚本

```shell
cd /mnt/deploy
ls
./springboot-start.sh stop
./springboot-start.sh start
```

springboot-start.sh

```shell
#!/bin/bash
#这里可替换为你自己的执行程序，其他代码无需更改
APP_NAME=gs-spring-boot-0.1.0.jar
source /etc/profile
#使用说明，用来提示输入参数
usage() {
    echo "Usage: sh 执行脚本.sh [start|stop|restart|status]"
    exit 1
}
#检查程序是否在运行
is_exist(){
  pid=`ps -ef|grep $APP_NAME|grep -v grep|awk '{print $2}' `
  #如果不存在返回1，存在返回0     
  if [ -z "${pid}" ]; then
   return 1
  else
    return 0
  fi
}

#启动方法
start(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "${APP_NAME} is already running. pid=${pid} ."
  else
    nohup java -jar $APP_NAME >> catalina.out 2>&1 &
  fi
}

#停止方法
stop(){
  is_exist
  if [ $? -eq "0" ]; then
    kill -9 $pid
  else
    echo "${APP_NAME} is not running"
  fi  
}

#输出运行状态
status(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "${APP_NAME} is running. Pid is ${pid}"
  else
    echo "${APP_NAME} is NOT running."
  fi
}

#重启
restart(){
  stop
  start
}

#根据输入参数，选择执行对应方法，不输入则执行使用说明
case "$1" in
  "start")
    start
    ;;
  "stop")
    stop
    ;;
  "status")
    status
    ;;
  "restart")
    restart
    ;;
  *)
    usage
    ;;
esac
```
