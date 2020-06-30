---
title: 体验seata分布式事务框架
date: 2020-02-24T10:06:42+08:00
draft: false

categories: ["环境搭建"]
---
引用https://github.com/seata/seata-samples/blob/master/springcloud-nacos-seata/README.md

使用工程为https://github.com/seata/seata-samples.git中springcloud-nacos-seata
<!--more-->
# springcloud-nacos-seata

**分布式事务组件seata的使用demo，AT模式，集成nacos、springboot、springcloud、mybatis-plus，数据库采用mysql5.7版本**

demo中使用的相关版本号，具体请看代码。如果搭建个人demo不成功，验证是否是由版本导致，由于目前这几个项目更新比较频繁，版本稍有变化便会出现许多奇怪问题

* seata 1.1.0
* spring-cloud-alibaba-seata 2.1.0.RELEASE
* spring-cloud-starter-alibaba-nacos-discovery  0.2.1.RELEASE
* springboot 2.0.6.RELEASE
* springcloud Finchley.RELEASE

----------

## 1. 服务端配置

### 1.1 Nacos-server
使用portainer拉取nacos并安装,使用默认端口
### 1.2 Seata-server
使用portainer拉取seataio/seata-server:latest安装

```shell script

# 将容器内的配置导出到宿主机，修改配置后启动容器，使容器使用宿主机的配置
docker cp b9e5dff571fe:/seata-server/resources /app/cloud/seata/conf
```

按照如下参数进行配置容器
```shell script
docker run --name  seata-server  \
        -d -p 8091:8091 \
        -e SEATA_CONFIG_NAME=file:/root/seata-config/registry  \
        -e SEATA_IP=192.168.1.2 \
        -v D://app/cloud/seata/conf:/root/seata-config  \
        -v D://app/cloud/seata/logs:/root/logs \
        seataio/seata-server
```
这里参考文章[SpringCloud Alibaba微服务实战九 - Seata 容器化](https://juejin.im/post/5e16e8fb5188254c0a040953)
#### 1.2.1 修改conf/registry.conf 配置

设置type、设置serverAddr为你的nacos节点地址。

**注意这里有一个坑，serverAddr不能带‘http://’前缀**

~~~java
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    serverAddr = "localhost"
    namespace = ""
    cluster = "default"
  }
}
config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"
  nacos {
    serverAddr = "localhost"
    namespace = ""
    cluster = "default"
  }
}

~~~

#### 1.2.2 修改conf/nacos-config.txt 配置

service.vgroupMapping.${your-service-gruop}=default，中间的${your-service-gruop}为自己定义的服务组名称，服务中的application.properties文件里配置服务组名称。

demo中有两个服务，分别是storage-service和order-service，所以配置如下

~~~shell

service.vgroupMapping.storage-service-group=default
service.vgroupMapping.order-service-group=default
~~~

#### 1.3 启动seata-server

**分两步，如下**

~~~shell
# 初始化seata的nacos配置
# 这个运行脚本在源码scrip/config-center/nacos路径下

cd scrip/config-center/nacos
sh nacos-config.sh 

# 启动seata-server
cd bin
sh seata-server.sh -p 8091 -m db
~~~

----------

## 2. 应用配置

### 2.1 数据库初始化

~~~SQL
-- 创建 order库、业务表、undo_log表
create database seata_order;
use seata_order;

DROP TABLE IF EXISTS `order_tbl`;
CREATE TABLE `order_tbl` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` varchar(255) DEFAULT NULL,
  `commodity_code` varchar(255) DEFAULT NULL,
  `count` int(11) DEFAULT 0,
  `money` int(11) DEFAULT 0,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `undo_log`
(
  `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT,
  `branch_id`     BIGINT(20)   NOT NULL,
  `xid`           VARCHAR(100) NOT NULL,
  `context`       VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB     NOT NULL,
  `log_status`    INT(11)      NOT NULL,
  `log_created`   DATETIME     NOT NULL,
  `log_modified`  DATETIME     NOT NULL,
  `ext`           VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;


-- 创建 storage库、业务表、undo_log表
create database seata_storage;
use seata_storage;

DROP TABLE IF EXISTS `storage_tbl`;
CREATE TABLE `storage_tbl` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `commodity_code` varchar(255) DEFAULT NULL,
  `count` int(11) DEFAULT 0,
  PRIMARY KEY (`id`),
  UNIQUE KEY (`commodity_code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `undo_log`
(
  `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT,
  `branch_id`     BIGINT(20)   NOT NULL,
  `xid`           VARCHAR(100) NOT NULL,
  `context`       VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB     NOT NULL,
  `log_status`    INT(11)      NOT NULL,
  `log_created`   DATETIME     NOT NULL,
  `log_modified`  DATETIME     NOT NULL,
  `ext`           VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;

-- 初始化库存模拟数据
INSERT INTO seata_storage.storage_tbl (id, commodity_code, count) VALUES (1, 'product-1', 9999999);
INSERT INTO seata_storage.storage_tbl (id, commodity_code, count) VALUES (2, 'product-2', 0);
~~~

### 2.2 应用配置

见代码

几个重要的配置

1. 每个应用的resource里需要配置一个registry.conf ，demo中与seata-server里的配置相同
2. application.propeties 的各个配置项，注意spring.cloud.alibaba.seata.tx-service-group 是服务组名称，与nacos-config.txt 配置的service.vgroup_mapping.${your-service-gruop}具有对应关系

----------

## 3. 测试

1. 分布式事务成功，模拟正常下单、扣库存

```shell script
localhost:9091/order/placeOrder/commit   
```


2. 分布式事务失败，模拟下单成功、扣库存失败，最终同时回滚
```shell script
localhost:9091/order/placeOrder/rollback 
```



## 4.遇到的问题
将seata中config.txt中的配置导入nacos，这个需要执行脚本，参考官方链接[README.md](https://github.com/seata/seata/blob/1.1.0/script/config-center/README.md)
部署过程中使用的mysql5.8版本，seata中mysql-connector-java为5.1.30版本，有兼容问题，所以建议使用mysql5.7版本运行

怎么说，seata这个框架还需要完善的地方，虽然1.0.0 GA版本发布，但是觉得bug还是蛮多。这个东西应该会越来越完善，值得持续关注与学习。

## 5.运行结果日志示例
### 运行出错
![](https://bj.bcebos.com/v1/alertcode-blog/体验seata分布式事务框架/seata-1.png)
### 运行成功
![](https://bj.bcebos.com/v1/alertcode-blog/体验seata分布式事务框架/seata-2.png)
