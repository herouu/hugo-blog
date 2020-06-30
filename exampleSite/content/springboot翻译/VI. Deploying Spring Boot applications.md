---
title: VI. Deploying Spring Boot applications
date: 2018-06-01 18:16:40
tags: ["springboot"]
categories: ["springboot翻译"]
---

### 部署到云端

对于大多数流行云 PaaS（平台即服务）提供商，Spring Boot 的可执行 jars 就是为它们准备的。这些提供商往往要求你自己提供容器，它们只负责管理应用的进程（不特别针对 Java 应用程序），所以它们需要一些中间层来将你的应用适配到云概念中的一个运行进程。

两个流行的云提供商，Heroku 和 Cloud Foundry，采取一个打包（'buildpack'）方法。为了启动你的应用程序，不管需要什么，buildpack 都会将它们打包到你的部署代码：它可能是一个 JDK 和一个 java 调用，也可能是一个内嵌的 webserver，或者是一个成熟的应用服务器。buildpack 是可插拔的，但你最好尽可能少的对它进行自定义设置。这可以减少不受你控制的功能范围，最小化部署和生产环境的发散。

理想情况下，你的应用就像一个 Spring Boot 可执行 jar，所有运行需要的东西都打包到它内部。

本章节我们将看到在“Getting Started”章节开发的简单应用是怎么在云端运行的。

<!--more-->

### 55.1 Cloud Foundry

如果不指定其他打包方式，Cloud Foundry 会启用它提供的默认打包方式。Cloud Foundry 的[Java buildpack](https://github.com/cloudfoundry/java-buildpack)对 Spring 应用有出色的支持，包括 Spring Boot。你可以部署独立的可执行 jar 应用，也可以部署传统的`.war`形式的应用。

一旦你构建应用（比如，使用`mvn clean package`）并[安装`cf`命令行工具](http://docs.cloudfoundry.org/devguide/installcf/install-go-cli.html)，你可以使用下面的`cf push`命令（将路径指向你编译后的`.jar`）来部署应用。在发布应用前，确保[你已登陆 cf 命令行客户端](http://docs.cloudfoundry.org/devguide/installcf/whats-new-v6.html#login)。

```shell
$ cf push acloudyspringtime -p target/demo-0.0.1-SNAPSHOT.jar
```

查看[`cf push`文档](http://docs.cloudfoundry.org/devguide/installcf/whats-new-v6.html#push)获取更多可选项。如果相同目录下存在[manifest.yml](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)，Cloud Foundry 会使用它。

就此，`cf`将开始上传你的应用：

```java
Uploading acloudyspringtime... OK
Preparing to start acloudyspringtime... OK
-----> Downloaded app package (8.9M)
-----> Java Buildpack source: system
-----> Downloading Open JDK 1.7.0_51 from .../x86_64/openjdk-1.7.0_51.tar.gz (1.8s)
       Expanding Open JDK to .java-buildpack/open_jdk (1.2s)
-----> Downloading Spring Auto Reconfiguration from  0.8.7 .../auto-reconfiguration-0.8.7.jar (0.1s)
-----> Uploading droplet (44M)
Checking status of app 'acloudyspringtime'...
  0 of 1 instances running (1 starting)
  ...
  0 of 1 instances running (1 down)
  ...
  0 of 1 instances running (1 starting)
  ...
  1 of 1 instances running (1 running)

App started
```

恭喜！应用现在处于运行状态！

检验部署应用的状态是很简单的：

```shell
$ cf apps
Getting applications in ...
OK

name                 requested state   instances   memory   disk   urls
...
acloudyspringtime    started           1/1         512M     1G     acloudyspringtime.cfapps.io
...
```

一旦 Cloud Foundry 意识到你的应用已经部署，你就可以点击给定的应用 URI，此处是[acloudyspringtime.cfapps.io/](http://acloudyspringtime.cfapps.io/)。

### 55.1.1 绑定服务

默认情况下，运行应用的元数据和服务连接信息被暴露为应用的环境变量（比如`$VCAP_SERVICES`），采用这种架构的原因是因为 Cloud Foundry 多语言特性（任何语言和平台都支持作为 buildpack），进程级别的环境变量是语言无关（language agnostic）的。

环境变量并不总是有利于设计最简单的 API，所以 Spring Boot 自动提取它们，然后将这些数据导入能够通过 Spring `Environment`抽象访问的属性里：

```java
@Component
class MyBean implements EnvironmentAware {

    private String instanceId;

    @Override
    public void setEnvironment(Environment environment) {
        this.instanceId = environment.getProperty("vcap.application.instance_id");
    }

    // ...

}
```

所有的 Cloud Foundry 属性都以`vcap`作为前缀，你可以使用 vcap 属性获取应用信息（比如应用的公共 URL）和服务信息（比如数据库证书），具体参考`CloudFoundryVcapEnvironmentPostProcessor` Javadoc。

**注**：[Spring Cloud Connectors](http://cloud.spring.io/spring-cloud-connectors/)项目很适合比如配置数据源的任务，Spring Boot 为它提供了自动配置支持和一个`spring-boot-starter-cloud-connectors` starter。

### Heroku

Heroku 是另外一个流行的 Paas 平台，你可以提供一个`Procfile`来定义 Heroku 的构建过程，它提供部署应用所需的指令。Heroku 为 Java 应用分配一个端口，确保能够路由到外部 URI。

你必须配置你的应用监听正确的端口，下面是用于我们的 starter REST 应用的`Procfile`：

```shell
web: java -Dserver.port=$PORT -jar target/demo-0.0.1-SNAPSHOT.jar
```

Spring Boot 将`-D`参数作为属性，通过 Spring `Environment`实例访问。`server.port`配置属性适合于内嵌的 Tomcat，Jetty 或 Undertow 实例启用时使用，`$PORT`环境变量被分配给 Heroku Paas 使用。

Heroku 默认使用 Java 1.8，只要你的 Maven 或 Gradle 构建时使用相同的版本就没问题（Maven 用户可以设置`java.version`属性）。如果你想使用 JDK 1.7，在你的`pom.xml`和`Procfile`临近处创建一个`system.properties`文件，在该文件中添加以下设置：

```java
java.runtime.version=1.7
```

这就是你需要做的所有内容，对于 Heroku 部署来说，经常做的工作就是使用`git push`将代码推送到生产环境。

```shell
$ git push heroku master

Initializing repository, done.
Counting objects: 95, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (78/78), done.
Writing objects: 100% (95/95), 8.66 MiB | 606.00 KiB/s, done.
Total 95 (delta 31), reused 0 (delta 0)

-----> Java app detected
-----> Installing OpenJDK 1.8... done
-----> Installing Maven 3.3.1... done
-----> Installing settings.xml... done
-----> executing /app/tmp/cache/.maven/bin/mvn -B
       -Duser.home=/tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229
       -Dmaven.repo.local=/app/tmp/cache/.m2/repository
       -s /app/tmp/cache/.m2/settings.xml -DskipTests=true clean install

       [INFO] Scanning for projects...
       Downloading: http://repo.spring.io/...
       Downloaded: http://repo.spring.io/... (818 B at 1.8 KB/sec)
        ....
       Downloaded: http://s3pository.heroku.com/jvm/... (152 KB at 595.3 KB/sec)
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/target/...
       [INFO] Installing /tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/pom.xml ...
       [INFO] ------------------------------------------------------------------------
       [INFO] BUILD SUCCESS
       [INFO] ------------------------------------------------------------------------
       [INFO] Total time: 59.358s
       [INFO] Finished at: Fri Mar 07 07:28:25 UTC 2014
       [INFO] Final Memory: 20M/493M
       [INFO] ------------------------------------------------------------------------

-----> Discovering process types
       Procfile declares types -> web

-----> Compressing... done, 70.4MB
-----> Launching... done, v6
       http://agile-sierra-1405.herokuapp.com/ deployed to Heroku

To git@heroku.com:agile-sierra-1405.git
 * [new branch]      master -> master

```

现在你的应用已经启动并运行在 Heroku。

### Openshift

[Openshift](https://www.openshift.com/)是 RedHat 公共（和企业）PaaS 解决方案。和 Heroku 相似，它也是通过运行被 git 提交触发的脚本来工作的，所以你可以使用任何你喜欢的方式编写 Spring Boot 应用启动脚本，只要 Java 运行时环境可用（这是在 Openshift 上可以要求的一个标准特性）。为了实现这样的效果，你可以使用[DIY Cartridge](https://www.openshift.com/developers/do-it-yourself)，并在`.openshift/action_scripts`下 hooks 你的仓库：

基本模式如下：

1.确保 Java 和构建工具已被远程安装，比如使用一个`pre_build` hook（默认会安装 Java 和 Maven，不会安装 Gradle）。

2.使用一个`build` hook 去构建你的 jar（使用 Maven 或 Gradle），比如：

```shell
#!/bin/bash
cd $OPENSHIFT_REPO_DIR
mvn package -s .openshift/settings.xml -DskipTests=true
```

3.添加一个调用`java -jar …`的`start` hook

```shell
#!/bin/bash
cd $OPENSHIFT_REPO_DIR
nohup java -jar target/*.jar --server.port=${OPENSHIFT_DIY_PORT} --server.address=${OPENSHIFT_DIY_IP} &
```

4.使用一个`stop` hook

```shell
#                  $OPENSHIFT_CARTRIDGE_SDK_BASH
PID=$(ps -ef | grep java.*\.jar | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    client_result "Application is already stopped"
else
    kill $PID
fi
```

5.将内嵌的服务绑定到平台提供的`application.properties`定义的环境变量，比如：

```shell
spring.datasource.url: jdbc:mysql://${OPENSHIFT_MYSQL_DB_HOST}:${OPENSHIFT_MYSQL_DB_PORT}/${OPENSHIFT_APP_NAME}
spring.datasource.username: ${OPENSHIFT_MYSQL_DB_USERNAME}
spring.datasource.password: ${OPENSHIFT_MYSQL_DB_PASSWORD}
```

在 Openshift 的网站上有一篇[running Gradle in Openshift](https://www.openshift.com/blogs/run-gradle-builds-on-openshift)博客，如果想使用 gradle 构建运行的应用可以参考它。

### 55.4 Boxfuse 和 Amazon Web Services

[Boxfuse](https://boxfuse.com/)的工作机制是将你的 Spring Boot 可执行 jar 或 war 转换进一个最小化的 VM 镜像，该镜像不需改变就能部署到 VirtualBox 或 AWS。Boxfuse 深度集成 Spring Boot 并使用你的 Spring Boot 配置文件自动配置端口和健康检查 URLs，它将该信息用于产生的镜像及它提供的所有资源（实例，安全分组，可伸缩的负载均衡等）。

一旦创建一个[Boxfuse account](https://console.boxfuse.com/)，并将它连接到你的 AWS 账号，安装最新版 Boxfuse 客户端，你就能按照以下操作将 Spring Boot 应用部署到 AWS（首先要确保应用被 Maven 或 Gradle 构建过，比如`mvn clean package`）：

```shell
$ boxfuse run myapp-1.0.jar -env=prod
```

更多选项可查看[`boxfuse run`文档](https://boxfuse.com/docs/commandline/run.html)，如果当前目录存在一个[boxfuse.conf](https://boxfuse.com/docs/commandline/#configuration)文件，Boxfuse 将使用它。

**注** 如果你的可执行 jar 或 war 包含[`application-boxfuse.properties`](https://boxfuse.com/docs/payloads/springboot.html#configuration)文件，Boxfuse 默认在启动时会激活一个名为`boxfuse`的 Spring profile，然后在该 profile 包含的属性基础上构建自己的配置。

此刻`boxfuse`将为你的应用创建一个镜像并上传到 AWS，然后配置并启动需要的资源：

```shell
Fusing Image for myapp-1.0.jar ...
Image fused in 00:06.838s (53937 K) -> axelfontaine/myapp:1.0
Creating axelfontaine/myapp ...
Pushing axelfontaine/myapp:1.0 ...
Verifying axelfontaine/myapp:1.0 ...
Creating Elastic IP ...
Mapping myapp-axelfontaine.boxfuse.io to 52.28.233.167 ...
Waiting for AWS to create an AMI for axelfontaine/myapp:1.0 in eu-central-1 (this may take up to 50 seconds) ...
AMI created in 00:23.557s -> ami-d23f38cf
Creating security group boxfuse-sg_axelfontaine/myapp:1.0 ...
Launching t2.micro instance of axelfontaine/myapp:1.0 (ami-d23f38cf) in eu-central-1 ...
Instance launched in 00:30.306s -> i-92ef9f53
Waiting for AWS to boot Instance i-92ef9f53 and Payload to start at http://52.28.235.61/ ...
Payload started in 00:29.266s -> http://52.28.235.61/
Remapping Elastic IP 52.28.233.167 to i-92ef9f53 ...
Waiting 15s for AWS to complete Elastic IP Zero Downtime transition ...
Deployment completed successfully. axelfontaine/myapp:1.0 is up and running at http://myapp-axelfontaine.boxfuse.io/
```

你的应用现在应该已经在 AWS 上启动并运行了。

这里有篇[在 EC2 部署 Spring Boot 应用](https://boxfuse.com/blog/spring-boot-ec2.html)的博客，Boxfuse 官网也有[Boxfuse 集成 Spring Boot 文档](https://boxfuse.com/docs/payloads/springboot.html)，你可以拿来作为参考。

### Google App Engine

Google App Engine 关联了 Servlet 2.5 API，如果不做一些修改你是不能在其上部署 Spring 应用的，具体查看本指南的[Servlet 2.5 章节](../IX. ‘How-to’ guides/81.5. Deploying a WAR in an Old (Servlet 2.5) Container.md)。

### 安装 Spring Boot 应用

除了使用`java -jar`运行 Spring Boot 应用，制作在 Unix 系统完全可执行的应用也是可能的，这会简化常见生产环境 Spring Boot 应用的安装和管理。在 Maven 中添加以下 plugin 配置可以创建一个"完全可执行"jar：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

对于 Gradle 等价的配置如下：

```shell
apply plugin: 'spring-boot'

springBoot {
    executable = true
}
```

然后输入`./my-application.jar`运行应用（`my-application`是你的 artifact name）。

**注** 完全可执行 jars 在文件前内嵌了一个额外脚本，目前不是所有工具都能接受这种形式，所以你有时可能不能使用该技术。

**注** 默认脚本支持大多数 Linux 分发版本，并在 CentOS 和 Ubuntu 上测试过。其他平台，比如 OS X 和 FreeBSD，可能需要使用自定义`embeddedLaunchScript`。

**注** 当一个完全可执行 jar 运行时，它会将 jar 的目录作为工作目录。

### 56.1 Unix/Linux 服务

你可以使用`init.d`或`systemd`启动 Spring Boot 应用，就像其他 Unix/Linux 服务那样。

### 安装为 init.d 服务(System V)

如果你配置 Spring Boot 的 Maven 或 Gradle 插件产生一个[完全可执行 jar](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#deployment-install)，并且没有使用自定义的`embeddedLaunchScript`，那你的应用可以作为`init.d`服务使用。只要简单的建立 jar 到`init.d`的符号连接就能获取标准的`start`，`stop`，`restart`和`status`命令支持。

该脚本支持以下特性：

- 以拥有该 jar 文件的用户启动服务。
- 使用`/var/run/<appname>/<appname>.pid`跟踪应用的 PID。
- 将控制台日志输出到`/var/log/<appname>.log`。

假设你在`/var/myapp`目录安装了一个 Spring Boot 应用，只需要建立符号连接就能将 Spring Boot 应用安装成`init.d`服务：

```shell
$ sudo ln -s /var/myapp/myapp.jar /etc/init.d/myapp
```

一旦安装成功，你就可以像平常那样启动和停止服务，例如，在一个基于 Debian 的系统：

```shell
$ service myapp start
```

**注** 如果应用启动失败，检查下`/var/log/<appname>.log`中的错误日志。

你也可以标识应用使用标准的操作系统工具自启动，例如，在 Debian 上：

```shell
$ update-rc.d myapp defaults <priority>
```

**保护 init.d 服务**

当使用`root`用户启动`init.d`服务时，默认的执行脚本将以拥有该 jar 文件的用户来运行应用。你最好不要使用`root`启动 Spring Boot 应用，也就是你的应用 jar 文件拥有者不能是`root`，而是创建一个特定用户运行应用，并使用`chown`指定该用户拥有 jar 文件，示例：

```shell
$ chown bootapp:bootapp your-app.jar
```

本示例中，默认执行脚本将使用`bootapp`用户运行应用。

**注** 为减少应用用户账号冲突，你可以考虑防止它使用登陆 shell，例如将账号 shell 设置为`/usr/sbin/nologin`。

你也要采取措施防止修改应用 jar 文件，首先配置 jar 文件权限只能被拥有者读取和执行，不能写入：

```shell
$ chmod 500 your-app.jar
```

然后，你也应该采取措施限制应用或账号运行时的冲突造成的损坏。如果攻击者获取访问权，他们可能会让 jar 文件可写并改变它的内容，使用`chattr`让它变为不可变是唯一的保护措施：

```shell
$ sudo chattr +i your-app.jar
```

这会防止任何用户修改 jar 文件，包括 root。

如果 root 用户用来控制应用服务，并且你使用[.conf 文件](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#deployment-script-customization-conf-file)自定义它的启动，该`.conf`文件将被 root 用户读取和评估，因此它也需要保护。使用`chmod`改变文件权限只能被拥有者读取，然后使用`chown`改变文件拥有者为 root：

```shell
$ chmod 400 your-app.conf
$ sudo chown root:root your-app.conf
```

### 安装为 Systemd 服务

Systemd 是 System V init 系统的继任者，很多现代 Linux 分发版本都在使用，尽管你可以继续使用`init.d`脚本，但使用`systemd` ‘service’脚本启动 Spring Boot 应用是有可能的。

假设你在`/var/myapp`目录下安装一个 Spring Boot 应用，为了将它安装为一个`systemd`服务，你需要按照以下示例创建一个脚本，比如命名为`myapp.service`，然后将它放到`/etc/systemd/system`目录下：

```shell
[Unit]
Description=myapp
After=syslog.target

[Service]
User=myapp
ExecStart=/var/myapp/myapp.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

**注** 记得根据你的应用改变`Description`，`User`和`ExecStart`字段。

注意跟作为`init.d`服务运行不同，使用`systemd`这种方式运行应用，PID 文件和控制台日志文件表现是不同的，必须在‘service’脚本配置正确的字段，具体参考[service unit configuration man page](http://www.freedesktop.org/software/systemd/man/systemd.service.html)。

使用以下命令标识应用自动在系统 boot 上启动：

```shell
$ systemctl enable myapp.service
```

具体详情可参考`man systemctl`。

### 自定义启动脚本

Maven 或 Gradle 插件生成的默认内嵌启动脚本可以通过很多方法自定义，对于大多数开发者，使用默认脚本和一些自定义通常就足够了。如果发现不能自定义需要的东西，你可以使用`embeddedLaunchScript`选项生成自己的文件。

**在脚本生成时自定义**

自定义写入 jar 文件的启动脚本元素是有意义的，例如，为`init.d`脚本提供`description`，既然知道这会展示到前端，你可能会在生成 jar 时提供它。

为了自定义写入的元素，你需要为 Spring Boot Maven 或 Gradle 插件指定`embeddedLaunchScriptProperties`选项。

以下是默认脚本支持的可代替属性：

| 名称                       | 描述                                                                                                                                                    |
| :------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `mode`                     | 脚本模式，默认为`auto`                                                                                                                                  |
| `initInfoProvides`         | 'INIT INFO'部分的`Provides`，对于 Gradle 默认为`spring-boot-application`，对于 Maven 默认为`${project.artifactId}`                                      |
| `initInfoShortDescription` | ‘INIT INFO’部分的`Short-Description`，对于 Gradle 默认为`Spring Boot Application`，对于 Maven 默认为`${project.name}`                                   |
| `initInfoDescription`      | “INIT INFO”部分的`Description`，对于 Gradle 默认为`Spring Boot Application`，对于 Maven 默认为`${project.description}`（失败会回退到`${project.name}`） |
| `initInfoChkconfig`        | “INIT INFO”部分的`chkconfig`，默认为`2345 99 01`                                                                                                        |
| `confFolder`               | `CONF_FOLDER`的默认值，默认为包含 jar 的文件夹                                                                                                          |
| `logFolder`                | `LOG_FOLDER`的默认值，只对`init.d`服务有效                                                                                                              |
| `pidFolder`                | `PID_FOLDER`的默认值，只对`init.d`服务有效                                                                                                              |
| `useStartStopDaemon`       | 如果`start-stop-daemon`命令可用，它会控制该实例，默认为`true`                                                                                           |

**在脚本运行时自定义**

对于需要在 jar 文件生成后自定义的项目，你可以使用环境变量或配置文件。

默认脚本支持以下环境变量：

| 变量                    | 描述                                                                                                                                                                                                                                                             |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MODE`                  | 操作的模式，默认值依赖于 jar 构建方式，通常为`auto`（意味着它会尝试通过检查它是否为`init.d`目录的软连接来推断这是不是一个 init 脚本）。你可以显式将它设置为`service`，这样`stop|start|status|restart`命令就可以工作了，或如果你只是想在前台运行该脚本那只需`run` |
| `USE_START_STOP_DAEMON` | 如果`start-stop-daemon`命令可用，它将被用来控制该实例，默认为`true`                                                                                                                                                                                              |
| `PID_FOLDER`            | pid 文件夹的根目录（默认为`/var/run`）                                                                                                                                                                                                                           |
| `LOG_FOLDER`            | 存放日志文件的文件夹（默认为`/var/log`）                                                                                                                                                                                                                         |
| `CONF_FOLDER`           | 读取`.conf`文件的文件夹                                                                                                                                                                                                                                          |
| `LOG_FILENAME`          | 存放于`LOG_FOLDER`的日志文件名（默认为`<appname>.log`）                                                                                                                                                                                                          |
| `APP_NAME`              | 应用名，如果 jar 运行自一个软连接，脚本会猜测它的应用名。如果不是软连接，或你想显式设置应用名，这就很有用了                                                                                                                                                      |
| `RUN_ARGS`              | 传递给程序的参数（Spring Boot 应用）                                                                                                                                                                                                                             |
| `JAVA_HOME`             | 默认使用`PATH`指定`java`的位置，但如果在`$JAVA_HOME/bin/java`有可执行文件，你可以通过该属性显式设置                                                                                                                                                              |
| `JAVA_OPTS`             | JVM 启动时传递的配置项                                                                                                                                                                                                                                           |
| `JARFILE`               | 在脚本启动没内嵌其内的 jar 文件时显式设置 jar 位置                                                                                                                                                                                                               |
| `DEBUG`                 | 如果 shell 实例的`-x`标识有设值，则你能轻松看到脚本的处理逻辑                                                                                                                                                                                                    |

**注** `PID_FOLDER`，`LOG_FOLDER`和`LOG_FILENAME`变量只对`init.d`服务有效。对于`systemd`等价的自定义方式是使用‘service’脚本。

如果`JARFILE`和`APP_NAME`出现异常，上面的设置可以使用一个`.conf`文件进行配置。该文件预期是放到跟 jar 文件临近的地方，并且名字相同，但后缀为`.conf`而不是`.jar`。例如，一个命名为`/var/myapp/myapp.jar`的 jar 将使用名为`/var/myapp/myapp.conf`的配置文件：

**myapp.conf**

```properties
JAVA_OPTS=-Xmx1024M
LOG_FOLDER=/custom/log/folder
```

**注** 如果不喜欢配置文件放到 jar 附近，你可以使用`CONF_FOLDER`环境变量指定文件的位置。

想要学习如何正确的保护文件可以参考[the guidelines for securing an init.d service.](the guidelines for securing an init.d service)。

### Microsoft Windows 服务

在 Window 上，你可以使用[winsw](https://github.com/kohsuke/winsw)启动 Spring Boot 应用。这里有个单独维护的[示例](https://github.com/snicoll-scratches/spring-boot-daemon)为你演示了怎么一步步为 Spring Boot 应用创建 Windows 服务。

### 接下来阅读什么

打开[Cloud Foundry](http://www.cloudfoundry.com/)，[Heroku](https://www.heroku.com/)，[OpenShift](https://www.openshift.com/)和[Boxfuse](https://boxfuse.com/)网站获取更多 Paas 能提供的特性信息。这里只提到 4 个比较流行的 Java PaaS 提供商，由于 Spring Boot 遵从基于云的部署原则，所以你也可以自由考虑其他提供商。

下章节将继续讲解[Spring Boot CLI](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#cli)，你也可以直接跳到[build tool plugins](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#build-tool-plugins)。
