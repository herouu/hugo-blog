---
title: II. Getting started
tags: ["springboot"]
date: 2018-06-15 18:16:40
categories: ["springboot翻译"]
---

### 10. Spring Boot 安装

Spring Boot 可以跟经典的 Java 开发工具（Eclipse，IntelliJ 等）一起使用或安装成一个命令行工具。不管怎样，你都需要安装[Java SDK v1.6 ](http://www.java.com/)或更高版本。在开始之前，你需要检查下当前安装的 Java 版本：

```shell
$ java -version
```

如果你是一个 Java 新手，或只是想体验一下 Spring Boot，你可能想先尝试[Spring Boot CLI](10.2. Installing the Spring Boot CLI.md)，否则继续阅读“经典”地安装指南。

**注**：尽管 Spring Boot 兼容 Java 1.6，如果可能的话，你应该考虑使用 Java 最新版本。

<!--more-->

### 10.1. 为 Java 开发者准备的安装指南

对于 java 开发者来说，使用 Spring Boot 就跟使用其他 Java 库一样，只需要在你的 classpath 下引入适当的`spring-boot-*.jar`文件。Spring Boot 不需要集成任何特殊的工具，所以你可以使用任何 IDE 或文本编辑器；同时，Spring Boot 应用也没有什么特殊之处，你可以像对待其他 Java 程序那样运行，调试它。

尽管可以拷贝 Spring Boot jars，但我们还是建议你使用支持依赖管理的构建工具，比如 Maven 或 Gradle。

### 10.1.1. Maven 安装

Spring Boot 兼容 Apache Maven 3.2 或更高版本。如果本地没有安装 Maven，你可以参考[maven.apache.org](http://maven.apache.org/)上的指南。

**注**：在很多操作系统上，可以通过包管理器来安装 Maven。OSX Homebrew 用户可以尝试`brew install maven`，Ubuntu 用户可以运行`sudo apt-get install maven`。

Spring Boot 依赖使用的 groupId 为`org.springframework.boot`。通常，你的 Maven POM 文件会继承`spring-boot-starter-parent`工程，并声明一个或多个[“Starter POMs”](../III. Using Spring Boot/13.4. Starter POMs.md)依赖。此外，Spring Boot 提供了一个可选的[Maven 插件](../VIII. Build tool plugins/58. Spring Boot Maven plugin.md)，用于创建可执行 jars。

下面是一个典型的 pom.xml 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.0.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <!-- Add Spring repositories -->
    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

**注**：`spring-boot-starter-parent`是使用 Spring Boot 的一种不错的方式，但它并不总是最合适的。有时你可能需要继承一个不同的父 POM，或只是不喜欢我们的默认配置，那你可以使用 import 作用域这种替代方案，具体查看[Section 13.2.2, “Using Spring Boot without the parent POM”](../III. Using Spring Boot/13.1.2. Using Spring Boot without the parent POM.md)。

### 10.1.2. Gradle 安装

Spring Boot 兼容 Gradle 1.12 或更高版本。如果本地没有安装 Gradle，你可以参考[www.gradle.org](http://www.gradle.org/)上的指南。

Spring Boot 的依赖可通过 groupId `org.springframework.boot`来声明。通常，你的项目将声明一个或多个[“Starter POMs”](../III. Using Spring Boot/13.4. Starter POMs.md)依赖。Spring Boot 提供了一个很有用的[Gradle 插件](../VIII. Build tool plugins/59. Spring Boot Gradle plugin.md)，可以用来简化依赖声明，创建可执行 jars。

**注**：当你需要构建项目时，Gradle Wrapper 提供一种给力的获取 Gradle 的方式。它是一小段脚本和库，跟你的代码一块提交，用于启动构建进程，具体参考[Gradle Wrapper](www.gradle.org/docs/current/userguide/gradle_wrapper.html)。

下面是一个典型的`build.gradle`文件：

```gradle
buildscript {
    repositories {
        jcenter()
        maven { url "http://repo.spring.io/snapshot" }
        maven { url "http://repo.spring.io/milestone" }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.4.0.BUILD-SNAPSHOT")
    }
}

apply plugin: 'java'
apply plugin: 'spring-boot'

jar {
    baseName = 'myproject'
    version =  '0.0.1-SNAPSHOT'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/snapshot" }
    maven { url "http://repo.spring.io/milestone" }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}

```

### 10.2. Spring Boot CLI 安装

Spring Boot CLI 是一个命令行工具，可用于快速搭建基于 Spring 的原型。它支持运行[Groovy](http://groovy.codehaus.org/)脚本，这也就意味着你可以使用类似 Java 的语法，但不用写很多的模板代码。

Spring Boot 不一定非要配合 CLI 使用，但它绝对是 Spring 应用取得进展的最快方式（你咋不飞上天呢？）。

### 10.2.1. 手动安装

Spring CLI 分发包可以从 Spring 软件仓库下载：

1. [spring-boot-cli-1.4.0.BUILD-SNAPSHOT-bin.zip](http://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/1.4.0.BUILD-SNAPSHOT/spring-boot-cli-1.4.0.BUILD-SNAPSHOT-bin.zip)
2. [spring-boot-cli-1.4.0.BUILD-SNAPSHOT-bin.tar.gz](http://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/1.3.0.BUILD-SNAPSHOT/spring-boot-cli-1.4.0.BUILD-SNAPSHOT-bin.tar.gz)

不稳定的[snapshot 分发包](http://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)也可以获取到。

下载完成后，解压分发包，根据存档里的[INSTALL.txt](http://raw.github.com/spring-projects/spring-boot/master/spring-boot-cli/src/main/content/INSTALL.txt)操作指南进行安装。总的来说，在`.zip`文件的`bin/`目录下会有一个 spring 脚本（Windows 下是`spring.bat`），或使用`java -jar`运行`lib/`目录下的`.jar`文件（该脚本会帮你确保 classpath 被正确设置）。

### 10.2.2. 使用 SDKMAN 安装

SDKMAN（软件开发包管理器）可以对各种各样的二进制 SDK 包进行版本管理，包括 Groovy 和 Spring Boot CLI。可以从[sdkman.io](http://sdkman.io/)下载 SDKMAN，并使用以下命令安装 Spring Boot：

```shell
$ sdk install springboot
$ spring --version
Spring Boot v1.4.0.BUILD-SNAPSHOT
```

如果你正在为 CLI 开发新的特性，并想轻松获取刚构建的版本，可以使用以下命令：

```shell
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-1.4.0.BUILD-SNAPSHOT-bin/spring-1.4.0.BUILD-SNAPSHOT/
$ sdk default springboot dev
$ spring --version
Spring CLI v1.4.0.BUILD-SNAPSHOT
```

这将会安装一个名叫 dev 的本地 spring 实例，它指向你的目标构建位置，所以每次你重新构建 Spring Boot，spring 都会更新为最新的。

你可以通过以下命令来验证：

```shell
$ sdk ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 1.4.0.BUILD-SNAPSHOT

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```

### 10.2.3. 使用 OSX Homebrew 进行安装

如果你的环境是 Mac，并使用[Homebrew](http://brew.sh/)，想要安装 Spring Boot CLI 只需以下操作：

```shell
$ brew tap pivotal/tap
$ brew install springboot
```

Homebrew 将把 spring 安装到`/usr/local/bin`下。

**注**：如果该方案不可用，可能是因为你的 brew 版本太老了。你只需执行`brew update`并重试即可。

### 10.2.4. 使用 MacPorts 进行安装

如果你的环境是 Mac，并使用[MacPorts](http://www.macports.org/)，想要安装 Spring Boot CLI 只需以下操作：

```shell
$ sudo port install spring-boot-cli
```

### 10.2.5. 命令行实现

Spring Boot CLI 启动脚本为[BASH](http://en.wikipedia.org/wiki/Bash_%28Unix_shell%29)和[zsh](http://en.wikipedia.org/wiki/Zsh) shells 提供完整的命令行实现。你可以在任何 shell 中 source 脚本（名称也是 spring），或将它放到用户或系统范围内的 bash 初始化脚本里。在 Debian 系统中，系统级的脚本位于`/shell-completion/bash`下，当新的 shell 启动时该目录下的所有脚本都会被执行。如果想要手动运行脚本，假如你已经安装了 SDKMAN，可以使用以下命令：

```shell
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
  grab  help  jar  run  test  version
```

**注**：如果你使用 Homebrew 或 MacPorts 安装 Spring Boot CLI，命令行实现脚本会自动注册到你的 shell。

### 10.2.6. Spring CLI 示例快速入门

下面是一个相当简单的 web 应用，你可以用它测试 Spring CLI 安装是否成功。创建一个名叫`app.groovy`的文件：

```groovy
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```

然后只需在 shell 中运行以下命令：

```shell
$ spring run app.groovy
```

**注**：首次运行该应用将会花费一些时间，因为需要下载依赖，后续运行将会快很多。

使用你最喜欢的浏览器打开[localhost:8080](localhost:8080)，然后就可以看到如下输出：

```java
Hello World!
```

### 10.3. 版本升级

如果你正在升级 Spring Boot 的早期发布版本，那最好查看下[project wiki](http://github.com/spring-projects/spring-boot/wiki)上的"release notes"，你会发现每次发布对应的升级指南和一个"new and noteworthy"特性列表。

想要升级一个已安装的 CLI，你需要使用合适的包管理命令，例如`brew upgrade`；如果是手动安装 CLI，按照[standard instructions](10.2.1. Manual installation.md)操作并记得更新你的 PATH 环境变量以移除任何老的引用。

### 11. 开发你的第一个 Spring Boot 应用

我们将使用 Java 开发一个简单的"Hello World" web 应用，以此强调下 Spring Boot 的一些关键特性。项目采用 Maven 进行构建，因为大多数 IDEs 都支持它。

**注**：[spring.io](http://spring.io/)网站包含很多 Spring Boot"入门"指南，如果你正在找特定问题的解决方案，可以先去那瞅瞅。你也可以简化下面的步骤，直接从[start.spring.io](https://start.spring.io/)的依赖搜索器选中`web` starter，这会自动生成一个新的项目结构，然后你就可以 happy 的敲代码了。具体详情参考[文档](https://github.com/spring-io/initializr)。

在开始前，你需要打开终端检查下安装的 Java 和 Maven 版本是否可用：

```shell
$ java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) 64-Bit Server VM (build 24.51-b03, mixed mode)
```

```shell
$ mvn -v
Apache Maven 3.2.3 (33f8c3e1027c3ddde99d3cdebad2656a31e8fdf4; 2014-08-11T13:58:10-07:00)
Maven home: /Users/user/tools/apache-maven-3.1.1
Java version: 1.7.0_51, vendor: Oracle Corporation
```

**注**：该示例需要创建单独的文件夹，后续的操作建立在你已创建一个合适的文件夹，并且它是你的“当前目录”。

### 11.1. 创建 POM

让我们以创建一个 Maven `pom.xml`文件作为开始吧，因为`pom.xml`是构建项目的处方！打开你最喜欢的文本编辑器，并添加以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Additional lines to be added here... -->

    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

这样一个可工作的构建就完成了，你可以通过运行`mvn package`测试它（暂时忽略"jar 将是空的-没有包含任何内容！"的警告）。

**注**：此刻，你可以将该项目导入到 IDE 中（大多数现代的 Java IDE 都包含对 Maven 的内建支持）。简单起见，我们将继续使用普通的文本编辑器完成该示例。

### 11.2. 添加 classpath 依赖

Spring Boot 提供很多"Starters"，用来简化添加 jars 到 classpath 的操作。示例程序中已经在 POM 的`parent`节点使用了`spring-boot-starter-parent`，它是一个特殊的 starter，提供了有用的 Maven 默认设置。同时，它也提供一个`dependency-management`节点，这样对于期望（”blessed“）的依赖就可以省略 version 标记了。

其他”Starters“只简单提供开发特定类型应用所需的依赖。由于正在开发 web 应用，我们将添加`spring-boot-starter-web`依赖-但在此之前，让我们先看下目前的依赖：

```shell
$ mvn dependency:tree
[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

`mvn dependency:tree`命令可以将项目依赖以树形方式展现出来，你可以看到`spring-boot-starter-parent`本身并没有提供依赖。编辑`pom.xml`，并在`parent`节点下添加`spring-boot-starter-web`依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

如果再次运行`mvn dependency:tree`，你将看到现在多了一些其他依赖，包括 Tomcat web 服务器和 Spring Boot 自身。

### 11.3. 编写代码

为了完成应用程序，我们需要创建一个单独的 Java 文件。Maven 默认会编译`src/main/java`下的源码，所以你需要创建那样的文件结构，并添加一个名为`src/main/java/Example.java`的文件：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```

尽管代码不多，但已经发生了很多事情，让我们分步探讨重要的部分吧！

### 11.3.1. @RestController 和@RequestMapping 注解

Example 类上使用的第一个注解是`@RestController`，这被称为构造型（stereotype）注解。它为阅读代码的人提供暗示（这是一个支持 REST 的控制器），对于 Spring，该类扮演了一个特殊角色。在本示例中，我们的类是一个 web `@Controller`，所以当 web 请求进来时，Spring 会考虑是否使用它来处理。

`@RequestMapping`注解提供路由信息，它告诉 Spring 任何来自"/"路径的 HTTP 请求都应该被映射到`home`方法。`@RestController`注解告诉 Spring 以字符串的形式渲染结果，并直接返回给调用者。

**注**：`@RestController`和`@RequestMapping`是 Spring MVC 中的注解（它们不是 Spring Boot 的特定部分），具体参考 Spring 文档的[MVC 章节](http://mvc.linesh.tw)。

### 11.3.2. @EnableAutoConfiguration 注解

第二个类级别的注解是`@EnableAutoConfiguration`，这个注解告诉 Spring Boot 根据添加的 jar 依赖猜测你想如何配置 Spring。由于`spring-boot-starter-web`添加了 Tomcat 和 Spring MVC，所以 auto-configuration 将假定你正在开发一个 web 应用，并对 Spring 进行相应地设置。

**Starters 和 Auto-Configuration**：Auto-configuration 设计成可以跟"Starters"一起很好的使用，但这两个概念没有直接的联系。你可以自由地挑选 starters 以外的 jar 依赖，Spring Boot 仍会尽最大努力去自动配置你的应用。

### 11.3.3. main 方法

应用程序的最后部分是 main 方法，这是一个标准的方法，它遵循 Java 对于一个应用程序入口点的约定。我们的 main 方法通过调用`run`，将业务委托给了 Spring Boot 的 SpringApplication 类。SpringApplication 将引导我们的应用，启动 Spring，相应地启动被自动配置的 Tomcat web 服务器。我们需要将`Example.class`作为参数传递给`run`方法，以此告诉 SpringApplication 谁是主要的 Spring 组件，并传递 args 数组以暴露所有的命令行参数。

### 11.4. 运行示例

到此，示例应用可以工作了。由于使用了`spring-boot-starter-parent` POM，这样我们就有了一个非常有用的 run 目标来启动程序。在项目根目录下输入`mvn spring-boot:run`启动应用：

```shell
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.4.1.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

如果使用浏览器打开[localhost:8080](http://localhost:8080)，你应该可以看到如下输出：

```shell
Hello World!
```

点击`ctrl-c`温雅地关闭应用程序。

### 11.5. 创建可执行 jar

让我们通过创建一个完全自包含，并可以在生产环境运行的可执行 jar 来结束示例吧！可执行 jars（有时被称为胖 jars "fat jars"）是包含编译后的类及代码运行所需依赖 jar 的存档。

**可执行 jars 和 Java**：Java 没有提供任何标准方式，用于加载内嵌 jar 文件（即 jar 文件中还包含 jar 文件），这对分发自包含应用来说是个问题。为了解决该问题，很多开发者采用"共享的"jars。共享的 jar 只是简单地将所有 jars 的类打包进一个单独的存档，这种方式存在的问题是，很难区分应用程序中使用了哪些库。在多个 jars 中如果存在相同的文件名（但内容不一样）也会是一个问题。Spring Boot 采取一个[不同的方式](../X. Appendices/D. The executable jar format.md)，允许你真正的直接内嵌 jars。

为了创建可执行的 jar，我们需要将`spring-boot-maven-plugin`添加到`pom.xml`中，在 dependencies 节点后面插入以下内容：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**注**：`spring-boot-starter-parent` POM 包含绑定到 repackage 目标的`<executions>`配置。如果不使用 parent POM，你需要自己声明该配置，具体参考[插件文档](http://docs.spring.io/spring-boot/docs/1.4.1.BUILD-SNAPSHOT/maven-plugin/usage.html)。

保存`pom.xml`，并从命令行运行`mvn package`：

```shell
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:1.4.1.BUILD-SNAPSHOT:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

如果查看 target 目录，你应该可以看到`myproject-0.0.1-SNAPSHOT.jar`，该文件大概有 10Mb。想查看内部结构，可以运行`jar tvf`：

```shell
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

在该目录下，你应该还能看到一个很小的名为`myproject-0.0.1-SNAPSHOT.jar.original`的文件，这是在 Spring Boot 重新打包前，Maven 创建的原始 jar 文件。

可以使用`java -jar`命令运行该应用程序：

```shell
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v1.3.0.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```

如上所述，点击`ctrl-c`可以温雅地退出应用。

### 12. 接下来阅读什么

希望本章节已为你提供一些 Spring Boot 的基础部分，并帮你找到开发自己应用的方式。如果你是任务驱动型的开发者，那可以直接跳到[spring.io](http://spring.io/)，check out 一些[入门指南](http://spring.io/guides/)，以解决特定的"使用 Spring 如何做"的问题；我们也有 Spring Boot 相关的[How-to](../IX. ‘How-to’ guides/README.md)参考文档。

[Spring Boot 仓库](http://github.com/spring-projects/spring-boot)有大量可以运行的[示例](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples)，这些示例代码是彼此独立的(运行或使用示例的时候不需要构建其他示例)。

否则，下一步就是阅读 [III、使用 Spring Boot](../III. Using Spring Boot/README.md)，如果没耐心，可以跳过该章节，直接阅读 [IV、Spring Boot 特性](../IV. Spring Boot features/README.md)。

### 8. Spring Boot 介绍

Spring Boot 简化了基于 Spring 的应用开发，你只需要"run"就能创建一个独立的，产品级别的 Spring 应用。
我们为 Spring 平台及第三方库提供开箱即用的设置，这样你就可以有条不紊地开始。多数 Spring Boot 应用只需要很少的 Spring 配置。

你可以使用 Spring Boot 创建 Java 应用，并使用`java -jar`启动它或采用传统的 war 部署方式。我们也提供了一个运行"spring 脚本"的命令行工具。

我们主要的目标是：

- 为所有 Spring 开发提供一个从根本上更快，且随处可得的入门体验。
- 开箱即用，但通过不采用默认设置可以快速摆脱这种方式。
- 提供一系列大型项目常用的非功能性特征，比如：内嵌服务器，安全，指标，健康检测，外部化配置。
- 绝对没有代码生成，也不需要 XML 配置。

### 9. 系统要求

默认情况下，Spring Boot 1.4.0.BUILD-SNAPSHOT 需要[Java7](http://www.java.com/)环境，Spring 框架 4.3.2.BUILD-SNAPSHOT 或以上版本。你可以在 Java6 下使用 Spring Boot，不过需要添加额外配置。具体参考[Section 82.11, “How to use Java 6” ](../IX. ‘How-to’ guides/73.9. How to use Java 6.md)。明确提供构建支持的有 Maven（3.2+）和 Gradle（1.12+）。

**注**：尽管你可以在 Java6 或 Java7 环境下使用 Spring Boot，通常建议尽可能使用 Java8。

### 9.1. Servlet 容器

下列内嵌容器支持开箱即用（out of the box）：

| 名称         | Servlet 版本 | Java 版本 |
| ------------ | :----------- | :-------- |
| Tomcat 8     | 3.1          | Java 7+   |
| Tomcat 7     | 3.0          | Java 6+   |
| Jetty 9.3    | 3.1          | Java 8+   |
| Jetty 9.2    | 3.1          | Java 7+   |
| Jetty 8      | 3.0          | Java 6+   |
| Undertow 1.3 | 3.1          | Java 7+   |

你也可以将 Spring Boot 应用部署到任何兼容 Servlet 3.0+的容器。
