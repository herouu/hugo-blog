---
title: VII. Spring Boot CLI
date: 2018-06-01 18:16:40
tags: ["springboot"]
categories: ["springboot翻译"]
---

### 58. 安装 CLI

你可以手动安装 Spring Boot CLI，也可以使用 SDKMAN!（SDK 管理器）或 Homebrew，MacPorts（如果你是一个 OSX 用户），具体安装指令参考"Getting started"的[Section 10.2, “Installing the Spring Boot CLI” ](../II. Getting started/10.2. Installing the Spring Boot CLI.md)章节。

<!--more-->

### 59. 使用 CLI

一旦安装好 CLI，你可以输入`spring`来运行它。如果不使用任何参数运行`spring`，将会展现一个简单的帮助界面：

```shell
$ spring
usage: spring [--help] [--version]
       <command> [<args>]

Available commands are:

  run [options] <files> [--] [args]
    Run a spring groovy script

  ... more command help is shown here
```

你可以使用`help`获取任何支持命令的详细信息，例如：

```shell
$ spring help run
spring run - Run a spring groovy script

usage: spring run [options] <files> [--] [args]

Option                     Description
------                     -----------
--autoconfigure [Boolean]  Add autoconfigure compiler
                             transformations (default: true)
--classpath, -cp           Additional classpath entries
-e, --edit                 Open the file with the default system
                             editor
--no-guess-dependencies    Do not attempt to guess dependencies
--no-guess-imports         Do not attempt to guess imports
-q, --quiet                Quiet logging
-v, --verbose              Verbose logging of dependency
                             resolution
--watch                    Watch the specified file for changes
```

`version`命令提供一个检查你正在使用的 Spring Boot 版本的快速方式：

```shell
$ spring version
Spring CLI v1.4.1.RELEASE
```

### 59.1 使用 CLI 运行应用

你可以使用`run`命令编译和运行 Groovy 源代码。Spring Boot CLI 完全自包含，以致于你不需要安装任何外部的 Groovy。

下面是一个使用 Groovy 编写的"hello world" web 应用：

hello.grooy

```java
@RestController
class WebApplication {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```

编译和运行应用可以输入：

```shell
$ spring run hello.groovy
```

你可以使用`--`将命令行参数和"spring"命令参数区分开来，例如：

```shell
$ spring run hello.groovy -- --server.port=9000
```

你可以使用`JAVA_OPTS`环境变量设置 JVM 命令行参数，例如：

```shell
$ JAVA_OPTS=-Xmx1024m spring run hello.groovy
```

### 推断"grab"依赖

标准的 Groovy 包含一个`@Grab`注解，它允许你声明对第三方库的依赖。这项有用的技术允许 Groovy 以和 Maven 或 Gradle 相同的方式下载 jars，但不需要使用构建工具。

Spring Boot 进一步延伸了该技术，它会基于你的代码尝试推导你"grab"哪个库。例如，由于`WebApplication`代码上使用了`@RestController`注解，"Tomcat"和"Spring MVC"将被获取（grabbed）。

下面 items 被用作"grab hints"：

| items                                                    | Grabs                         |
| -------------------------------------------------------- | :---------------------------- |
| `JdbcTemplate`,`NamedParameterJdbcTemplate`,`DataSource` | JDBC 应用                     |
| `@EnableJms`                                             | JMS 应用                      |
| `@EnableCaching`                                         | 缓存抽象                      |
| `@Test`                                                  | JUnit                         |
| `@EnableRabbit`                                          | RabbitMQ                      |
| `@EnableReactor`                                         | 项目重构                      |
| extends `Specification`                                  | Spock test                    |
| `@EnableBatchProcessing`                                 | Spring Batch                  |
| `@MessageEndpoint`,`@EnableIntegrationPatterns`          | Spring 集成                   |
| `@EnableDeviceResolver`                                  | Spring Mobile                 |
| `@Controller`,`@RestController`,`@EnableWebMvc`          | Spring MVC + 内嵌 Tomcat      |
| `@EnableWebSecurity`                                     | Spring Security               |
| `@EnableTransactionManagement`                           | Spring Transaction Management |

**注** 想要理解自定义是如何生效的，可以查看 Spring Boot CLI 源码中的[CompilerAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-cli/src/main/java/org/springframework/boot/cli/compiler/CompilerAutoConfiguration.java)子类。

### 推断"grab"坐标

Spring Boot 扩展了 Groovy 标准`@Grab`注解，使其能够允许你指定一个没有`group`或`version`的依赖，例如`@Grab('freemarker')`。Spring Boot 使用默认依赖元数据推断 artifact’s 的 group 和 version，需要注意的是默认元数据和你使用的 CLI 版本有绑定关系－只有在迁移到新版本的 CLI 时它才会改变，这样你就可以控制何时改变依赖了，在[附录](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#appendix-dependency-versions)的表格中可以查看默认元数据包含的依赖和它们的版本。

### 默认 import 语句

为了帮助你减少 Groovy 代码量，一些`import`语句被自动包含进来了。注意上面的示例中引用`@Component`，`@RestController`和`@RequestMapping`而没有使用全限定名或`import`语句。

**注**：很多 Spring 注解在不使用`import`语句的情况下可以正常工作。尝试运行你的应用，看一下在添加 imports 之前哪些会失败。

### 自动创建 main 方法

跟等效的 Java 应用不同，你不需要在 Groovy 脚本中添加一个`public static void main(String[] args)`方法。Spring Boot 会使用你编译后的代码自动创建一个`SpringApplication`。

### 59.1.5 自定义依赖管理

默认情况下，CLI 使用在解析`@Grab`依赖时`spring-boot-dependencies`声明的依赖管理，其他的依赖管理会覆盖默认的依赖管理，并可以通过`@DependencyManagementBom`注解进行配置。该注解的值必须是一个或多个 Maven BOMs 的候选（`groupId:artifactId:version`）。

例如，以下声明：

```grovy
@DependencyManagementBom("com.example.custom-bom:1.0.0")
```

将选择 Maven 仓库中`com/example/custom-versions/1.0.0/`下的`custom-bom-1.0.0.pom`。

当指定多个 BOMs 时，它们会以声明次序进行应用，例如：

```grovy
@DependencyManagementBom(["com.example.custom-bom:1.0.0",
        "com.example.another-bom:1.0.0"])
```

意味着`another-bom`的依赖将覆盖`custom-bom`依赖。

能够使用`@Grab`的地方，你同样可以使用`@DependencyManagementBom`。然而，为了确保依赖管理的一致次序，你在应用中至多使用一次`@DependencyManagementBom`。[Spring IO Platform](http://platform.spring.io/)是一个非常有用的依赖元数据源(Spring Boot 的超集)，例如：
`@DependencyManagementBom('io.spring.platform:platform-bom:1.1.2.RELEASE')`。

### 59.2 测试你的代码

`test`命令允许你编译和运行应用程序的测试用例，常规使用方式如下：

```shell
$ spring test app.groovy tests.groovy
Total: 1, Success: 1, : Failures: 0
Passed? true
```

在这个示例中，`test.groovy`包含 JUnit `@Test`方法或 Spock `Specification`类。所有的普通框架注解和静态方法在不使用`import`导入的情况下，仍旧可以使用。

下面是我们使用的`test.groovy`文件（含有一个 JUnit 测试）：

```java
class ApplicationTests {

    @Test
    void homeSaysHello() {
        assertEquals("Hello World!", new WebApplication().home())
    }

}
```

**注** 如果有多个测试源文件，你可能倾向于将它们放到`test`目录下。

### 59.3 多源文件应用

你可以在所有接收文件输入的命令中使用 shell 通配符。这允许你轻松处理来自一个目录下的多个文件，例如：

```shell
$ spring run *.groovy
```

如果想将`test`或`spec`代码从主应用代码中分离，这项技术就十分有用了：

```shell
$ spring test app/*.groovy test/*.groovy
```

### 59.4 应用打包

你可以使用`jar`命令打包应用程序为一个可执行的 jar 文件，例如：

```shell
$ spring jar my-app.jar *.groovy
```

最终的 jar 包括编译应用产生的类和所有依赖，这样你就可以使用`java -jar`来执行它了。该 jar 文件也包含了来自应用 classpath 的实体。你可以使用`--include`和`--exclude`添加明确的路径（两者都是用逗号分割，同样都接收值为'+'和'-'的前缀，'-'意味着它们将从默认设置中移除），默认包含（includes）：

```shell
public/**, resources/**, static/**, templates/**, META-INF/**, *
```

默认排除(excludes)：

```shell
.*, repository/**, build/**, target/**, **/*.jar, **/*.groovy
```

查看`spring help jar`可以获得更多信息。

### 59.5 初始化新工程

`init`命令允许你使用[start.spring.io](https://start.spring.io/)在不离开 shell 的情况下创建一个新的项目，例如：

```shell
$ spring init --dependencies=web,data-jpa my-project
Using service at https://start.spring.io
Project extracted to '/Users/developer/example/my-project'
```

这创建了一个`my-project`目录，它是一个基于 Maven 且依赖`spring-boot-starter-web`和`spring-boot-starter-data-jpa`的项目。你可以使用`--list`参数列出该服务的能力。

```shell
$ spring init --list
=======================================
Capabilities of https://start.spring.io
=======================================

Available dependencies:
-----------------------
actuator - Actuator: Production ready features to help you monitor and manage your application
...
web - Web: Support for full-stack web development, including Tomcat and spring-webmvc
websocket - Websocket: Support for WebSocket development
ws - WS: Support for Spring Web Services

Available project types:
------------------------
gradle-build -  Gradle Config [format:build, build:gradle]
gradle-project -  Gradle Project [format:project, build:gradle]
maven-build -  Maven POM [format:build, build:maven]
maven-project -  Maven Project [format:project, build:maven] (default)

...
```

`init`命令支持很多选项，查看`help`输出可以获得更多详情。例如，下面的命令创建一个使用 Java8 和打包为`war`的 gradle 项目：

```shell
$ spring init --build=gradle --java-version=1.8 --dependencies=websocket --packaging=war sample-app.zip
Using service at https://start.spring.io
Content saved to 'sample-app.zip'
```

### 59.6 使用内嵌 shell

Spring Boot 包括完整的 BASH 和 zsh shells 的命令行脚本，如果这两种你都不使用（可能你是一个 Window 用户），那你可以使用`shell`命令启用一个集成 shell。

```shell
$ spring shell
Spring Boot (v1.4.1.RELEASE)
Hit TAB to complete. Type \'help' and hit RETURN for help, and \'exit' to quit.
```

从内嵌 shell 中可以直接运行其他命令：

```shell
$ version
Spring CLI v1.4.1.RELEASE
```

内嵌 shell 支持 ANSI 彩色输出和 tab 补全，如果需要运行一个原生命令，你可以使用`!`前缀，点击`ctrl-c`将退出内嵌 shell。

### 59.7 为 CLI 添加扩展

使用`install`命令可以为 CLI 添加扩展，该命令接收一个或多个格式为`group:artifact:version`的 artifact 坐标集，例如：

```shell
$ spring install com.example:spring-boot-cli-extension:1.0.0.RELEASE
```

除安装你提供坐标的 artifacts 标识外，该 artifacts 的所有依赖也会被安装。

使用`uninstall`可以卸载一个依赖，和`install`命令一样，它也接收一个或多个格式为`group:artifact:version`的 artifact 坐标集，例如：

```shell
$ spring uninstall com.example:spring-boot-cli-extension:1.0.0.RELEASE
```

它会通过你提供的坐标卸载相应的 artifacts 标识及它们的依赖。

为了卸载所有附加依赖，你可以使用`--all`选项，例如：

```shell
$ spring uninstall --all
```

### 60. 使用 Groovy beans DSL 开发应用

Spring 框架 4.0 版本对`beans{}`"DSL"（借鉴自[Grails](http://grails.org/)）提供原生支持，你可以使用相同格式在 Groovy 应用程序脚本中嵌入 bean 定义。有时这是引入外部特性的很好方式，比如中间件声明，例如：

```java
@Configuration
class Application implements CommandLineRunner {

    @Autowired
    SharedService service

    @Override
    void run(String... args) {
        println service.message
    }

}

import my.company.SharedService

beans {
    service(SharedService) {
        message = "Hello World"
    }
}
```

你可以使用`beans{}`混合位于相同文件的类声明，只要它们都处于顶级，或如果喜欢的话，你可以将 beans DSL 放到一个单独的文件中。

### 61. 使用 settings.xml 配置 CLI

Spring Boot CLI 使用 Maven 的依赖解析引擎 Aether 来解析依赖，它充分利用发现的`~/.m2/settings.xml` Maven 设置去配置 Aether。

CLI 支持以下配置：

- Offline
- Mirrors
- Servers
- Proxies
- Profiles
  - Activation

  - Repositories
- Active profiles

更多信息可参考[Maven 设置文档](https://maven.apache.org/settings.html)。

### 62. 接下来阅读什么

GitHub 仓库有一些[groovy 脚本示例](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-cli/samples)可用于尝试 Spring Boot CLI，[源码](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-cli/src/main/java/org/springframework/boot/cli)里也有丰富的文档说明。

如果发现已触及 CLI 工具的限制，你可以将应用完全转换为 Gradle 或 Maven 构建的 groovy 工程。下一章节将覆盖 Spring Boot 的[构建工具](../VIII. Build tool plugins/README.md)，这些工具可以跟 Gradle 或 Maven 一起使用。
