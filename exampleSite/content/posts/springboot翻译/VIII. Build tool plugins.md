---
title: VIII. Build tool plugins
date: 2018-06-01 18:16:40
tags: ["springboot"]
categories: ["springboot翻译"]
---

### 63. Spring Boot Maven 插件

[Spring Boot Maven 插件](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/)为 Maven 提供 Spring Boot 支持，它允许你打包可执行 jar 或 war 存档，然后就地运行应用。为了使用它，你需要使用 Maven 3.2（或更高版本）。

**注** 参考[Spring Boot Maven Plugin Site](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/)可以获取全部的插件文档。

<!--more-->

### 63.1 包含该插件

想要使用 Spring Boot Maven 插件只需简单地在你的 pom.xml 的`plugins`部分包含相应的 XML：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- ... -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>1.4.1.RELEASE</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

该配置会在 Maven 生命周期的`package`阶段重新打包一个 jar 或 war。下面的示例展示在`target`目录下既有重新打包后的 jar，也有原始的 jar：

```shell
$ mvn package
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```

如果不包含像上面那样的`<execution/>`，你可以自己运行该插件（但只有在 package 目标也被使用的情况），例如：

```shell
$ mvn package spring-boot:repackage
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```

如果使用一个里程碑或快照版本，你还需要添加正确的`pluginRepository`元素：

```xml
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
```

### 63.2 打包可执行 jar 和 war 文件

一旦`spring-boot-maven-plugin`被包含到你的`pom.xml`中，Spring Boot 就会自动尝试使用`spring-boot:repackage`目标重写存档以使它们能够执行。为了构建一个 jar 或 war，你应该使用常规的`packaging`元素配置你的项目：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>jar</packaging>
    <!-- ... -->
</project>
```

生成的存档在`package`阶段会被 Spring Boot 增强。你想启动的 main 类即可以通过指定一个配置选项，也可以通过为 manifest 添加一个`Main-Class`属性这种常规的方式实现。如果你没有指定一个 main 类，该插件会搜索带有`public static void main(String[] args)`方法的类。

为了构建和运行一个项目的 artifact，你可以输入以下命令：

```shell
$ mvn package
$ java -jar target/mymodule-0.0.1-SNAPSHOT.jar
```

为了构建一个即可执行，又能部署到外部容器的 war 文件，你需要标记内嵌容器依赖为"provided"，例如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>war</packaging>
    <!-- ... -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <!-- ... -->
    </dependencies>
</project>
```

**注** 具体参考[“Section 81.1, “Create a deployable war file”” ](../IX. ‘How-to’ guides/81.1. Create a deployable war file.md)章节。

高级配置选项和示例可在[插件信息页面](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/)获取。

### 64. Spring Boot Gradle 插件

Spring Boot Gradle 插件为 Gradle 提供 Spring Boot 支持，它允许你打包可执行 jar 或 war 存档，运行 Spring Boot 应用，使用`spring-boot-dependencies`提供的依赖管理。

### 64.1 包含该插件

想要使用 Spring Boot Gradle 插件，你只需简单的包含一个`buildscript`依赖，并应用`spring-boot`插件：

```gradle
buildscript {
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.4.1.RELEASE")
    }
}
apply plugin: 'spring-boot'
```

如果使用的是一个里程碑或快照版本，你需要添加相应的`repositories`引用：

```gradle
buildscript {
    repositories {
        maven.url "http://repo.spring.io/snapshot"
        maven.url "http://repo.spring.io/milestone"
    }
    // ...
}
```

### 64.2 Gradle 依赖管理

`spring-boot`插件自动应用[Dependency Management Plugin](https://github.com/spring-gradle-plugins/dependency-management-plugin/)，并配置它导入`spring-boot-starter-parent` bom。这提供了跟 Maven 用户喜欢的相似依赖管理体验，例如，如果声明的依赖在 bom 中被管理的话，你就可以省略版本。为了充分使用该功能，只需要想通常那样声明依赖，但将版本号设置为空：

```gradle
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.thymeleaf:thymeleaf-spring4")
    compile("nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect")
}
```

**注** 你声明的`spring-boot` Gradle 插件的版本决定了`spring-boot-starter-parent` bom 导入的版本（确保可以重复构建）。你最好将`spring-boot` gradle 插件版本跟 Spring Boot 版本保持一致，版本详细信息可以在[附录](../X. Appendices/E. Dependency versions.md)中查看。

`spring-boot`插件对于没有指定版本的依赖只会提供一个版本。如果不想使用插件提供的版本，你可以像平常那样在声明依赖的时候指定版本。例如：

```gradle
dependencies {
    compile("org.thymeleaf:thymeleaf-spring4:2.1.1.RELEASE")
}
```

### 64.3 打包可执行 jar 和 war 文件

一旦`spring-boot`插件被应用到你的项目，它将使用`bootRepackage`任务自动尝试重写存档以使它们能够执行。为了构建一个 jar 或 war，你需要按通常的方式配置项目。

你想启动的 main 类既可以通过一个配置选项指定，也可以通过向 manifest 添加一个`Main-Class`属性。如果你没有指定 main 类，该插件会搜索带有`public static void main(String[] args)`方法的类。

为了构建和运行一个项目 artifact，你可以输入以下内容：

```shell
$ gradle build
$ java -jar build/libs/mymodule-0.0.1-SNAPSHOT.jar
```

为了构建一个即能执行也可以部署到外部容器的 war 包，你需要将内嵌容器依赖标记为`providedRuntime`，比如：

```gradle
...
apply plugin: 'war'

war {
    baseName = 'myapp'
    version =  '0.5.0'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/libs-snapshot" }
}

configurations {
    providedRuntime
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")
    ...
}
```

**注** 具体参考[“Section 81.1, “Create a deployable war file””](../IX. ‘How-to’ guides/81.1. Create a deployable war file.md)。

### 64.4 就地（in-place）运行项目

为了在不先构建 jar 的情况下运行项目，你可以使用`bootRun`任务：

```shell
$ gradle bootRun
```

如果项目中添加了[devtools](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-devtools)，它将自动监控你的应用变动。此外，你可以运行应用，这样静态 classpath 资源（比如，默认位于`src/main/resources`下）在应用运行期间将能够重新加载，这在开发期间是非常有用的：

```gradle
bootRun {
    addResources = true
}
```

让静态 classpath 资源可加载意味着`bootRun`不使用`processResources`任务的输出，例如，当使用`bootRun`调用时，你的应用将以未经处理的形式使用资源。

### 64.5 Spring Boot 插件配置

Gradle 插件自动扩展你的构建脚本 DSL，它为脚本添加一个`springBoot`元素以此作为 Boot 插件的全局配置。你可以像配置其他 Gradle 扩展那样为`springBoot`设置相应的属性（下面有配置选项列表）。

```gradle
springBoot {
    backupSource = false
}
```

### 64.6 Repackage 配置

该插件添加了一个`bootRepackage`任务，你可以直接配置它，比如：

```gradle
bootRepackage {
    mainClass = 'demo.Application'
}
```

下面是可用的配置选项：

| 名称                             | 描述                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enabled`                        | 布尔值，用于控制 repackager 的开关（如果你只想要 Boot 的其他特性而不是这个，那它就派上用场了）                                                                                                                                                                                                                                                                         |
| `mainClass`                      | 要运行的 main 类。如果没有指定，则使用 project 属性`mainClassName`。如果该应用插件没有使用或没有定义`mainClassName`，则搜索存档以寻找一个合适的类。"合适"意味着一个唯一的，具有良好格式的`main()`方法的类（如果找到多个则构建会失败）。你也可以通过`run`任务（`main`属性）指定`main`类的名称，和/或将"startScripts"（`mainClassName`属性）作为"springBoot"配置的替代。 |
| `classifier`                     | 添加到存档的一个文件名字段（在扩展之前），这样最初保存的存档仍旧存放在最初的位置。在存档被重新打包（repackage）的情况下，该属性默认为`null`。默认值适用于多数情况，但如果你想在另一个项目中使用原 jar 作为依赖，最好使用一个扩展来定义该可执行 jar                                                                                                                     |
| `withJarTask`                    | Jar 任务的名称或值，用于定位要被 repackage 的存档                                                                                                                                                                                                                                                                                                                      |
| `customConfiguration`            | 自定义配置的名称，用于填充内嵌的 lib 目录（不指定该属性，你将获取所有编译和运行时依赖）                                                                                                                                                                                                                                                                                |
| `executable`                     | 布尔值标识，表示 jar 文件在类 Unix 系统上是否完整可执行，默认为`false`                                                                                                                                                                                                                                                                                                 |
| `embeddedLaunchScript`           | 如果 jar 是完整可执行的，该内嵌启动脚本将添加到 jar。如果没有指定，将使用 Spring Boot 默认的脚本                                                                                                                                                                                                                                                                       |
| `embeddedLaunchScriptProperties` | 启动脚本暴露的其他属性，默认脚本支持`mode`属性，值可以是`auto`，`service`或`run`                                                                                                                                                                                                                                                                                       |
| `excludeDevtools`                | 布尔值标识，表示 devtools jar 是否应该从重新打包的存档中排除出去，默认为`false`                                                                                                                                                                                                                                                                                        |

### 64.7 使用 Gradle 自定义配置进行 Repackage

有时候不打包解析自`compile`，`runtime`和`provided`作用域的默认依赖可能更合适些。如果创建的可执行 jar 被原样运行，你需要将所有的依赖内嵌进该 jar 中；然而，如果目的是 explode 一个 jar 文件，并手动运行 main 类，你可能在`CLASSPATH`下已经有一些可用的库了。在这种情况下，你可以使用不同的依赖集重新打包（repackage）你的 jar。

使用自定义的配置将自动禁用来自`compile`，`runtime`和`provided`作用域的依赖解析。自定义配置即可以定义为全局的（处于`springBoot`部分内），也可以定义为任务级的。

```gradle
task clientJar(type: Jar) {
    appendix = 'client'
    from sourceSets.main.output
    exclude('**/*Something*')
}

task clientBoot(type: BootRepackage, dependsOn: clientJar) {
    withJarTask = clientJar
    customConfiguration = "mycustomconfiguration"
}
```

在以上示例中，我们创建了一个新的`clientJar` Jar 任务从你编译后的源中打包一个自定义文件集。然后我们创建一个新的`clientBoot` BootRepackage 任务，并让它使用`clientJar`任务和`mycustomconfiguration`。

```gradle
configurations {
    mycustomconfiguration.exclude group: 'log4j'
}

dependencies {
    mycustomconfiguration configurations.runtime
}
```

在`BootRepackage`中引用的配置是一个正常的[Gradle 配置](http://www.gradle.org/docs/current/dsl/org.gradle.api.artifacts.Configuration.html)。在以上示例中，我们创建了一个新的名叫`mycustomconfiguration`的配置，指示它来自一个`runtime`，并排除对`log4j`的依赖。如果`clientBoot`任务被执行，重新打包的 jar 将含有所有来自`runtime`作用域的依赖，除了`log4j` jars。

### 64.7.1 配置选项

可用的配置选项如下：

| 名称                    | 描述                                                                                                                                                                                                                      |
| ----------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `mainClass`             | 可执行 jar 运行的 main 类                                                                                                                                                                                                 |
| `providedConfiguration` | provided 配置的名称（默认为`providedRuntime`）                                                                                                                                                                            |
| `backupSource`          | 在重新打包之前，原先的存档是否备份（默认为`true`）                                                                                                                                                                        |
| `customConfiguration`   | 自定义配置的名称                                                                                                                                                                                                          |
| `layout`                | 存档类型，对应于内部依赖是如何制定的（默认基于存档类型进行推测），具体查看[available layouts](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#build-tool-plugins-gradle-configuration-layouts) |
| `requiresUnpack`        | 一个依赖列表（格式为"groupId:artifactId"，为了运行，它们需要从 fat jars 中解压出来。）所有节点被打包进胖 jar，但运行的时候它们将被自动解压                                                                                |

### 可用的 layouts

`layout`属性用于配置存档格式及启动加载器是否包含，以下为可用的 layouts：

| 名称               | 描述                                                                                                                                                                                                                 | 可执行 |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----- |
| `JAR`              | 常规的可执行[JAR layout](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#executable-jar-jar-file-structure)                                                                               | 是     |
| `WAR`              | 可执行[WAR layout](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#executable-jar-war-file-structure)，`provided`依赖放置到`WEB-INF/lib-provided`，以免`war`部署到 servlet 容器时造成冲突 | 是     |
| `ZIP`（别名`DIR`） | 跟`JAR` layout 类似，使用[PropertiesLauncher](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#                                                                                            |

### 64.8 理解 Gradle 插件是如何工作的

当`spring-boot`应用到你的 Gradle 项目，一个默认的名叫`bootRepackage`的任务被自动创建。`bootRepackage`任务依赖于 Gradle `assemble`任务，当执行时，它会尝试找到所有限定符为空的 jar artifacts（也就是说，tests 和 sources jars 被自动跳过）。

由于`bootRepackage`会查找'所有'创建的 jar artifacts，Gradle 任务执行的顺序就非常重要了。多数项目只创建一个单一的 jar 文件，所以通常这不是一个问题。然而，如果你正打算创建一个更复杂的，使用自定义`jar`和`BootRepackage`任务的项目 setup，有几个方面需要考虑。

如果'仅仅'从项目创建自定义 jar 文件，你可以简单地禁用默认的`jar`和`bootRepackage`任务：

```gradle
jar.enabled = false
bootRepackage.enabled = false
```

另一个选项是指示默认的`bootRepackage`任务只能使用一个默认的`jar`任务：

```gradle
bootRepackage.withJarTask = jar
```

如果你有一个默认的项目 setup，在该项目中，主（main）jar 文件被创建和重新打包。并且，你仍旧想创建额外的自定义 jars，你可以将自定义的 repackage 任务结合起来，然后使用`dependsOn`，这样`bootJars`任务就会在默认的`bootRepackage`任务执行以后运行：

```gradle
task bootJars
bootJars.dependsOn = [clientBoot1,clientBoot2,clientBoot3]
build.dependsOn(bootJars)
```

上面所有方面经常用于避免一个已经创建的 boot jar 又被重新打包的情况。重新打包一个存在的 boot jar 不是什么大问题，但你可能会发现它包含不必要的依赖。

### 64.9 使用 Gradle 将 artifacts 发布到 Maven 仓库

如果声明依赖但没有指定版本，且想要将 artifacts 发布到一个 Maven 仓库，那你需要使用详细的 Spring Boot 依赖管理来配置 Maven 发布。通过配置它发布继承自`spring-boot-starter-parent`的 poms 或引入来自`spring-boot-dependencies`的依赖管理可以实现该需求。这种配置的具体细节取决于你如何使用 Gradle 及如何发布该 artifacts。

### 64.9.1 自定义 Gradle，用于产生一个继承依赖管理的 pom

下面示例展示了如何配置 Gradle 去产生一个继承自`spring-boot-starter-parent`的 pom，更多信息请参考[Gradle 用户指南](http://gradle.org/docs/current/userguide/userguide.html)。

```gradle
uploadArchives {
    repositories {
        mavenDeployer {
            pom {
                project {
                    parent {
                        groupId "org.springframework.boot"
                        artifactId "spring-boot-starter-parent"
                        version "1.4.1.RELEASE"
                    }
                }
            }
        }
    }
}
```

### 64.9.2 自定义 Gradle，用于产生一个导入依赖管理的 pom

以下示例展示了如何配置 Gradle 产生一个导入`spring-boot-dependencies`提供的依赖管理的 pom，更多信息请参考[Gradle 用户指南](http://gradle.org/docs/current/userguide/userguide.html)。

```gradle
uploadArchives {
    repositories {
        mavenDeployer {
            pom {
                project {
                    dependencyManagement {
                        dependencies {
                            dependency {
                                groupId "org.springframework.boot"
                                artifactId "spring-boot-dependencies"
                                version "1.4.1.RELEASE"
                                type "pom"
                                scope "import"
                            }
                        }
                    }
                }
            }
        }
    }
}
```

### Spring Boot AntLib 模块

Spring Boot AntLib 模块为 Apache Ant 提供基本的 Spring Boot 支持，你可以使用该模块创建可执行的 jars。在`build.xml`添加额外的`spring-boot`命名空间就可以使用该模块了：

```xml
<project xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">
    ...
</project>
```

你需要记得在启动 Ant 时使用`-lib`选项，例如：

```shell
$ ant -lib <folder containing spring-boot-antlib-1.4.1.RELEASE.jar>
```

**注** 详细示例可参考[using Apache Ant with `spring-boot-antlib`
](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-ant)。

### Spring Boot Ant 任务

一旦声明`spring-boot-antlib`命名空间，以下任务就可用了。

### 65.1.2. 示例

**指定 start-class**

```xml
<spring-boot:exejar destfile="target/my-application.jar"
        classes="target/classes" start-class="com.foo.MyApplication">
    <resources>
        <fileset dir="src/main/resources" />
    </resources>
    <lib>
        <fileset dir="lib" />
    </lib>
</spring-boot:exejar>
```

**探测 start-class**

```xml
<exejar destfile="target/my-application.jar" classes="target/classes">
    <lib>
        <fileset dir="lib" />
    </lib>
</exejar>
```

### 示例

**查找并记录**

```xml
<findmainclass classesroot="target/classes" />
```

**查找并设置**

```xml
<findmainclass classesroot="target/classes" property="main-class" />
```

**覆盖并设置**

```xml
<findmainclass mainclass="com.foo.MainClass" property="main-class" />
```

### 66. 对其他构建系统的支持

如果想使用除了 Maven 和 Gradle 之外的构建工具，你可能需要开发自己的插件。可执行 jars 需要遵循一个特定格式，并且一些实体需要以不压缩的方式写入（详情查看附录中的[可执行 jar 格式](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#executable-jar)章节）。

Spring Boot Maven 和 Gradle 插件在实际生成 jars 的过程中会使用`spring-boot-loader-tools`，如果需要，你也可以自由地使用该 library。

### 66.1. 重新打包存档

使用`org.springframework.boot.loader.tools.Repackager`可以将一个存在的存档重新打包，这样它就变成一个自包含的可执行存档。`Repackager`类需要提供单一的构造器参数，该参数指向一个存在的 jar 或 war 包。你可以使用两个可用的`repackage()`方法中的一个来替换原始的文件或写入新的目标，在 repackager 运行前还可以指定各种配置。

### 66.2. 内嵌库

当重新打包一个存档时，你可以使用`org.springframework.boot.loader.tools.Libraries`接口来包含对依赖文件的引用。在这里我们不提供任何该`Libraries`接口的具体实现，因为它们通常跟具体的构建系统相关。

如果存档已经包含 libraries，你可以使用`Libraries.NONE`。

### 66.3. 查找 main 类

如果你没有使用`Repackager.setMainClass()`指定一个 main 类，该 repackager 将使用[ASM](http://asm.ow2.org/)去读取 class 文件，然后尝试查找一个合适的，具有`public static void main(String[] args)`方法的类。如果发现多个候选者，将会抛出异常。

### 66.4. repackage 实现示例

这是一个典型的 repackage 示例：

```java
Repackager repackager = new Repackager(sourceJarFile);
repackager.setBackupSource(false);
repackager.repackage(new Libraries() {
            @Override
            public void doWithLibraries(LibraryCallback callback) throws IOException {
                // Build system specific implementation, callback for each dependency
                // callback.library(new Library(nestedFile, LibraryScope.COMPILE));
            }
        });
```

### 67. 接下来阅读什么

如果对构建工具插件如何工作感兴趣，你可以查看 GitHub 上的[spring-boot-tools](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-tools)模块，附加中有详细的[可执行 jar 格式](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-tools)。

如果有特定构建相关的问题，可以查看[how-to](../IX. ‘How-to’ guides/README.md)指南。
