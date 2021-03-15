---
title: "Maven依赖第三方jar包引用的几种方式"
subtitle: ""
date: 2021-03-15T20:43:42+08:00
lastmod: 2021-03-15T20:43:42+08:00
draft: false
author: ""
authorLink: ""
description: ""

tags: ['devtools']
categories: ['devtools']

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
maven引用一些第三方jar的几种方式
<!--more-->
### 1. 上传到maven私服

### 2. install到本地

### 3. 使用local repository

```shell
mvn deploy:deploy-file -Durl=file://${basedir}/lib/ -Dfile=C:\Users\22696\Desktop\demo-0.0.1
-SNAPSHOT.jar -DartifactId=demo -Dpackaging=jar -Dversion=0.0.1-SNAPSHOT -DgroupId=com.example

```

* pom.xml中引用

```xml

<repositories>
    <repository>
        <id>my-local-repo</id>
        <name>My Repository</name>
        <url>file://${basedir}/lib/</url>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
        </releases>
    </repository>
</repositories>
```

* 依赖

```xml

<dependency>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0</version>
</dependency>
```

### 4. jar包放入lib目录下,工程中直接添加依赖

```xml

<dependency>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0</version>
    <systemPath>${basedir}/lib/demo-0.0.1-SNAPSHOT.jar</systemPath>
    <scope>system</scope>
</dependency>
```

### 5. 可以使用site-maven-plugin在github上搭建公有仓库

* 参考 https://segmentfault.com/a/1190000022756193