---
title: III. Using Spring Boot
tags: ["springboot"]
date: 2018-06-15 18:16:40
categories: ["springboot翻译"]
---
### 13. 构建系统

强烈建议你选择一个支持依赖管理，能消费发布到"Maven中央仓库"的artifacts的构建系统，比如Maven或Gradle。使用其他构建系统也是可以的，比如Ant，但它们可能得不到很好的支持。
<!--more-->
###      依赖管理

Spring Boot每次发布时都会提供一个它所支持的精选依赖列表。实际上，在构建配置里你不需要提供任何依赖的版本，因为Spring Boot已经替你管理好了。当更新Spring Boot时，那些依赖也会一起更新。

**注** 如果有必要，你可以指定依赖的版本来覆盖Spring Boot默认版本。

精选列表包括所有能够跟Spring Boot一起使用的Spring模块及第三方库，该列表可以在[材料清单(spring-boot-dependencies)](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-maven-without-a-parent)获取到，也可以找到一些支持[Maven](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-maven-parent-pom)和[Gradle](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#                                                            Spring Boot每次发布都关联一个Spring框架的基础版本，所以强烈建议你不要自己指定Spring版本。

### 13.2. Maven

Maven用户可以继承`spring-boot-starter-parent`项目来获取合适的默认设置。该parent项目提供以下特性：
- 默认编译级别为Java 1.6
- 源码编码为UTF-8
- 一个[Dependency management](./13.1. Dependency management.md)节点，允许你省略常见依赖的`<version>`标签，继承自`spring-boot-dependencies` POM。
- 恰到好处的[资源过滤](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)
- 恰到好处的插件配置（[exec插件](http://mojo.codehaus.org/exec-maven-plugin/)，[surefire](http://maven.apache.org/surefire/maven-surefire-plugin/)，[Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin)，[shade](http://maven.apache.org/plugins/maven-shade-plugin/)）
- 恰到好处的对`application.properties`和`application.yml`进行筛选，包括特定profile（profile-specific）的文件，比如`application-foo.properties`和`application-foo.yml`

最后一点：由于配置文件默认接收Spring风格的占位符（`${...}`），所以Maven filtering需改用`@..@`占位符（你可以使用Maven属性`resource.delimiter`来覆盖它）。

### 13.2.1. 继承starter parent

如果你想配置项目，让其继承自`spring-boot-starter-parent`，只需将`parent`按如下设置：
```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.1.BUILD-SNAPSHOT</version>
</parent>
```
**注**：你应该只需在该依赖上指定Spring Boot版本，如果导入其他的starters，放心的省略版本号好了。

按照以上设置，你可以在自己的项目中通过覆盖属性来覆盖个别的依赖。例如，你可以将以下设置添加到`pom.xml`中来升级Spring Data到另一个发布版本。
```xml
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

**注** 查看[spring-boot-dependencies pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-dependencies/pom.xml)获取支持的属性列表。

### 13.2.2. 在不使用parent POM的情况下玩转Spring Boot

不是每个人都喜欢继承`spring-boot-starter-parent` POM，比如你可能需要使用公司的标准parent，或只是倾向于显式声明所有的Maven配置。

如果你不想使用`spring-boot-starter-parent`，通过设置`scope=import`的依赖，你仍能获取到依赖管理的好处：
```xml
<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.4.1.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

以上设置不允许你使用属性覆盖个别依赖，为了达到这个目的，你需要在项目的`dependencyManagement`节点中，在`spring-boot-dependencies`实体前插入一个节点。例如，为了将Spring Data升级到另一个发布版本，你需要将以下配置添加到`pom.xml`中：
```xml
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.4.1.BUILD-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
**注** 示例中，我们指定了一个BOM，但任何的依赖类型都可以通过这种方式覆盖。

### 13.2.3. 改变Java版本

`spring-boot-starter-parent`选择了相当保守的Java兼容策略，如果你遵循我们的建议，使用最新的Java版本，可以添加一个`java.version`属性：
```xml
<properties>
    <java.version>1.8</java.version>
</properties>
```

### 13.2.4. 使用Spring Boot Maven插件

Spring Boot包含一个[Maven插件](../VIII. Build tool plugins/58. Spring Boot Maven plugin.md)，它可以将项目打包成一个可执行jar。如果想使用它，你可以将该插件添加到`<plugins>`节点处：
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
**注**：如果使用Spring Boot starter parent pom，你只需添加该插件而无需配置它，除非你想改变定义在partent中的设置。

### 13.3. Gradle

Gradle用户可以直接在它们的`dependencies`节点处导入”starters“。跟Maven不同的是，这里不用导入"super parent"，也就不能共享配置。
```gradle
apply plugin: 'java'

repositories {
    maven { url "http://repo.spring.io/snapshot" }
    maven { url "http://repo.spring.io/milestone" }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.4.1.BUILD-SNAPSHOT")
}
```
跟maven类似，spring boot也有gradle插件[spring-boot-gradle-plugin](../VIII. Build tool plugins/65. Spring Boot Gradle plugin.md)，它能够提供任务用于创建可执行jar，或从源码（source）运行项目。它也提供[依赖管理](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-gradle-dependency-management)的能力，该功能允许你省略Spring Boot管理的任何依赖的version版本号：
```gradle
buildscript {
    repositories {
        maven { url "http://repo.spring.io/snapshot" }
        maven { url "http://repo.spring.io/milestone" }
    }

    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.4.1.BUILD-SNAPSHOT")
    }
}

apply plugin: 'java'
apply plugin: 'spring-boot'

repositories {
    maven { url "http://repo.spring.io/snapshot" }
    maven { url "http://repo.spring.io/milestone" }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

### 13.4. Ant

使用Apache Ant+Ivy构建Spring Boot项目是完全可能的。`spring-boot-antlib` AntLib模块能够帮助Ant创建可执行jars，一个传统的用于声明依赖的`ivy.xml`文件可能如下所示：
```xml
<ivy-module version="2.0">
    <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
    <configurations>
        <conf name="compile" description="everything needed to compile this module" />
        <conf name="runtime" extends="compile" description="everything needed to run this module" />
    </configurations>
    <dependencies>
        <dependency org="org.springframework.boot" name="spring-boot-starter"
            rev="${spring-boot.version}" conf="compile" />
    </dependencies>
</ivy-module>
```
同样，一个传统的`build.xml`可能是这样的：
```xml
<project
    xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">

    <property name="spring-boot.version" value="1.3.0.BUILD-SNAPSHOT" />

    <target name="resolve" description="--> retrieve dependencies with ivy">
        <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
    </target>

    <target name="classpaths" depends="resolve">
        <path id="compile.classpath">
            <fileset dir="lib/compile" includes="*.jar" />
        </path>
    </target>

    <target name="init" depends="classpaths">
        <mkdir dir="build/classes" />
    </target>

    <target name="compile" depends="init" description="compile">
        <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
    </target>

    <target name="build" depends="compile">
        <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
            <spring-boot:lib>
                <fileset dir="lib/runtime" />
            </spring-boot:lib>
        </spring-boot:exejar>
    </target>
</project>
```

**注** 如果你不想使用`spring-boot-antlib`模块，那查看[Section 81.10, “Build an executable archive from Ant without using spring-boot-antlib”](../IX. ‘How-to’ guides/73.8. Build an executable archive with Ant.md)获取更多指导。

### 13.5. Starters

Starters是一个依赖描述符的集合，你可以将它包含进项目中，这样添加依赖就非常方便。你可以获取所有Spring及相关技术的一站式服务，而不需要翻阅示例代码，拷贝粘贴大量的依赖描述符。例如，如果你想使用Spring和JPA进行数据库访问，只需要在项目中包含`spring-boot-starter-data-jpa`依赖，然后你就可以开始了。

该starters包含很多搭建，快速运行项目所需的依赖，并提供一致的，可管理传递性的依赖集。

**名字有什么含义**：所有官方starters遵循相似的命名模式：`spring-boot-starter-*`，在这里`*`是一种特殊的应用程序类型。该命名结构旨在帮你找到需要的starter。很多集成于IDEs中的Maven插件允许你通过名称name搜索依赖。例如，使用相应的Eclipse或STS插件，你可以简单地在POM编辑器中点击`ctrl-space`，然后输入"spring-boot-starter"就可以获取一个完整列表。正如[Creating your own starter](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-custom-starter)章节中讨论的，第三方starters不应该以`spring-boot`开头，因为它跟Spring Boot官方artifacts冲突。一个acme的第三方starter通常命名为`acme-spring-boot-starter`。

以下应用程序starters是Spring Boot在`org.springframework.boot` group下提供的：

**表 13.1. Spring Boot application starters**

|名称|描述|Pom|
|------|:-----|:-----|
|spring-boot-starter-test|用于测试Spring Boot应用，支持常用测试类库，包括JUnit, Hamcrest和Mockito|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-test/pom.xml)|
|spring-boot-starter-mobile|用于使用Spring Mobile开发web应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-mobile/pom.xml)|
|spring-boot-starter-social-twitter|对使用Spring Social Twitter的支持|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-social-twitter/pom.xml)|
|spring-boot-starter-cache|用于使用Spring框架的缓存支持|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-cache/pom.xml)|
|spring-boot-starter-activemq|用于使用Apache ActiveMQ实现JMS消息|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-activemq/pom.xml)|
|spring-boot-starter-jta-atomikos|用于使用Atomikos实现JTA事务|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml)|
|spring-boot-starter-aop|用于使用Spring AOP和AspectJ实现面向切面编程|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-aop/pom.xml)|
|spring-boot-starter-web|用于使用Spring MVC构建web应用，包括RESTful。Tomcat是默认的内嵌容器|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-web/pom.xml)|
|spring-boot-starter-data-elasticsearch|用于使用Elasticsearch搜索，分析引擎和Spring Data Elasticsearch|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml)|
|spring-boot-starter-jdbc|对JDBC的支持（使用Tomcat JDBC连接池）|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-jdbc/pom.xml)|
|spring-boot-starter-batch|对Spring Batch的支持|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-batch/pom.xml)|
|spring-boot-starter-social-facebook|用于使用Spring Social Facebook|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-social-facebook/pom.xml)|
|spring-boot-starter-web-services|对Spring Web服务的支持|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-web-services/pom.xml)|
|spring-boot-starter-jta-narayana|Spring Boot Narayana JTA Starter|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-jta-narayana/pom.xml)|
|spring-boot-starter-thymeleaf|用于使用Thymeleaf模板引擎构建MVC web应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml)|
|spring-boot-starter-mail|用于使用Java Mail和Spring框架email发送支持|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-mail/pom.xml)|
|spring-boot-starter-jta-bitronix|用于使用Bitronix实现JTA事务|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml)|
|spring-boot-starter-data-mongodb|用于使用基于文档的数据库MongoDB和Spring Data MongoDB|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml)|
|spring-boot-starter-validation|用于使用Hibernate Validator实现Java Bean校验|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-validation/pom.xml)|
|spring-boot-starter-jooq|用于使用JOOQ访问SQL数据库，可使用[spring-boot-starter-data-jpa](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#spring-boot-starter-data-jpa)或[spring-boot-starter-jdbc](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#                                                                                                                                                                                            Data Redis和Jedis客户端操作键-值存储的Redis，在1.4中已被[spring-boot-starter-data-redis](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#spring-boot-starter-data-redis)取代|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-redis/pom.xml)|
|spring-boot-starter-data-cassandra|用于使用分布式数据库Cassandra和Spring Data Cassandra|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml)|
|spring-boot-starter-hateoas|用于使用Spring MVC和Spring HATEOAS实现基于超媒体的RESTful web应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-hateoas/pom.xml)|
|spring-boot-starter-integration|用于使用Spring Integration|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-integration/pom.xml)|
|spring-boot-starter-data-solr|通过Spring Data Solr使用Apache Solr搜索平台|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-solr/pom.xml)|
|spring-boot-starter-freemarker|用于使用FreeMarker模板引擎构建MVC web应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-freemarker/pom.xml)|
|spring-boot-starter-jersey|用于使用JAX-RS和Jersey构建RESTful web应用，可使用[spring-boot-starter-web](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#spring-boot-starter-web)替代|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-jersey/pom.xml)|
|spring-boot-starter|核心starter，包括自动配置支持，日志和YAML|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter/pom.xml)|
|spring-boot-starter-data-couchbase|用于使用基于文档的数据库Couchbase和Spring Data Couchbase|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml)|
|spring-boot-starter-artemis|使用Apache Artemis实现JMS消息|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-artemis/pom.xml)|
|spring-boot-starter-cloud-connectors|对Spring Cloud Connectors的支持，用于简化云平台下（例如Cloud Foundry 和Heroku）服务的连接|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml)|
|spring-boot-starter-social-linkedin|用于使用Spring Social LinkedIn|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-social-linkedin/pom.xml)|
|spring-boot-starter-velocity|用于使用Velocity模板引擎构建MVC web应用，**从1.4版本过期**|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-velocity/pom.xml)|
|spring-boot-starter-data-rest|用于使用Spring Data REST暴露基于REST的Spring Data仓库|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-rest/pom.xml)|
|spring-boot-starter-data-gemfire|用于使用分布式数据存储GemFire和Spring Data GemFire|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-gemfire/pom.xml)|
|spring-boot-starter-groovy-templates|用于使用Groovy模板引擎构建MVC web应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml)|
|spring-boot-starter-amqp|用于使用Spring AMQP和Rabbit MQ|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-amqp/pom.xml)|
|spring-boot-starter-hornetq|用于使用HornetQ实现JMS消息，被[spring-boot-starter-artemis](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#                                                                                                                                                                                               Web服务，被[spring-boot-starter-web-services](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#spring-boot-starter-web-services)取代|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-ws/pom.xml)|
|spring-boot-starter-security|对Spring Security的支持|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-security/pom.xml)|
|spring-boot-starter-data-redis|用于使用Spring Data Redis和Jedis客户端操作键—值数据存储Redis|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-redis/pom.xml)|
|spring-boot-starter-websocket|用于使用Spring框架的WebSocket支持构建WebSocket应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-websocket/pom.xml)|
|spring-boot-starter-mustache|用于使用Mustache模板引擎构建MVC web应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-mustache/pom.xml)|
|spring-boot-starter-data-neo4j|用于使用图数据库Neo4j和Spring Data Neo4j|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml)|
|spring-boot-starter-data-jpa|用于使用Hibernate实现Spring Data JPA|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml)|

除了应用程序starters，以下starters可用于添加[production ready](../V. Spring Boot Actuator/README.md)的功能：

**表 13.2. Spring Boot生产级starters**

|名称|描述|Pom|
|----|:----|:----|
|spring-boot-starter-actuator|用于使用Spring Boot的Actuator，它提供了production ready功能来帮助你监控和管理应用程序|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-actuator/pom.xml)|
|spring-boot-starter-remote-shell|用于通过SSH，使用CRaSH远程shell监控，管理你的应用|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-remote-shell/pom.xml)|

最后，Spring Boot还包含一些用于排除或交换某些特定技术方面的starters：

**表 13.3. Spring Boot技术性starters**

|名称|描述|Pom|
|------|:------|:------|
|spring-boot-starter-undertow|用于使用Undertow作为内嵌servlet容器，可使用[spring-boot-starter-tomcat](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#spring-boot-starter-tomcat)替代|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-undertow/pom.xml)|
|spring-boot-starter-logging|用于使用Logback记录日志，默认的日志starter|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-logging/pom.xml)|
|spring-boot-starter-tomcat|用于使用Tomcat作为内嵌servlet容器，[spring-boot-starter-web](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#                                                                                                                                                                                                                                                                                                                                  #spring-boot-starter-tomcat)替代|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-jetty/pom.xml)|
|spring-boot-starter-log4j2|用于使用Log4j2记录日志，可使用[spring-boot-starter-logging](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#spring-boot-starter-logging)代替|[Pom](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/spring-boot-starter-log4j2/pom.xml)|

**注**：查看GitHub上位于`spring-boot-starters`模块内的[README文件](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-starters/README.adoc)，可以获取到一个社区贡献的其他starters列表。

### 14. 组织你的代码

Spring Boot不要求使用任何特殊的代码结构，不过，遵循以下的一些最佳实践还是挺有帮助的。

### 14.1. 使用"default"包

当类没有声明`package`时，它被认为处于`default package`下。通常不推荐使用`default package`，因为对于使用`@ComponentScan`，`@EntityScan`或`@SpringBootApplication`注解的Spring Boot应用来说，它会扫描每个jar中的类，这会造成一定的问题。

**注** 我们建议你遵循Java推荐的包命名规范，使用一个反转的域名（例如`com.example.project`）。

### 14.2. 放置应用的main类

通常建议将应用的main类放到其他类所在包的顶层(root package)，并将`@EnableAutoConfiguration`注解到你的main类上，这样就隐式地定义了一个基础的包搜索路径（search package），以搜索某些特定的注解实体（比如@Service，@Component等）
。例如，如果你正在编写一个JPA应用，Spring将搜索`@EnableAutoConfiguration`注解的类所在包下的`@Entity`实体。

采用root package方式，你就可以使用`@ComponentScan`注解而不需要指定`basePackage`属性，也可以使用`@SpringBootApplication`注解，只要将main类放到root package中。

下面是一个典型的结构：
```shell
com
 +- example
     +- myproject
         +- Application.java
         |
         +- domain
         |   +- Customer.java
         |   +- CustomerRepository.java
         |
         +- service
         |   +- CustomerService.java
         |
         +- web
             +- CustomerController.java
```
`Application.java`将声明`main`方法，还有基本的`@Configuration`。
```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

### 15. 配置类

Spring Boot提倡基于Java的配置。尽管你可以使用XML源调用`SpringApplication.run()`，不过还是建议你使用`@Configuration`类作为主要配置源。通常定义了`main`方法的类也是使用`@Configuration`注解的一个很好的替补。

**注**：虽然网络上有很多使用XML配置的Spring示例，但你应该尽可能的使用基于Java的配置，搜索查看`enable*`注解就是一个好的开端。

### 15.1. 导入其他配置类

你不需要将所有的`@Configuration`放进一个单独的类，`@Import`注解可以用来导入其他配置类。另外，你也可以使用`@ComponentScan`注解自动收集所有Spring组件，包括`@Configuration`类。

### 15.2. 导入XML配置

如果必须使用XML配置，建议你仍旧从一个`@Configuration`类开始，然后使用`@ImportResource`注解加载XML配置文件。

### 16. 自动配置

Spring Boot自动配置（auto-configuration）尝试根据添加的jar依赖自动配置你的Spring应用。例如，如果classpath下存在`HSQLDB`，并且你没有手动配置任何数据库连接的beans，那么Spring Boot将自动配置一个内存型（in-memory）数据库。

实现自动配置有两种可选方式，分别是将`@EnableAutoConfiguration`或`@SpringBootApplication`注解到`@Configuration`类上。

**注**：你应该只添加一个`@EnableAutoConfiguration`注解，通常建议将它添加到主配置类（primary `@Configuration`）上。

### 16.1. 逐步替换自动配置

自动配置（Auto-configuration）是非侵入性的，任何时候你都可以定义自己的配置类来替换自动配置的特定部分。例如，如果你添加自己的`DataSource`  bean，默认的内嵌数据库支持将不被考虑。

如果需要查看当前应用启动了哪些自动配置项，你可以在运行应用时打开`--debug`开关，这将为核心日志开启debug日志级别，并将自动配置相关的日志输出到控制台。

### 16.2. 禁用特定的自动配置项

如果发现启用了不想要的自动配置项，你可以使用`@EnableAutoConfiguration`注解的exclude属性禁用它们：
```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```
如果该类不在classpath中，你可以使用该注解的excludeName属性，并指定全限定名来达到相同效果。最后，你可以通过`spring.autoconfigure.exclude`属性exclude多个自动配置项（一个自动配置项集合）。

**注** 通过注解级别或exclude属性都可以定义排除项。

### 17. Spring Beans和依赖注入

你可以自由地使用任何标准的Spring框架技术去定义beans和它们注入的依赖。简单起见，我们经常使用`@ComponentScan`注解搜索beans，并结合`@Autowired`构造器注入。

如果遵循以上的建议组织代码结构（将应用的main类放到包的最上层，即root package），那么你就可以添加`@ComponentScan`注解而不需要任何参数，所有应用组件（`@Component`, `@Service`, `@Repository`, `@Controller`等）都会自动注册成Spring Beans。

下面是一个`@Service` Bean的示例，它使用构建器注入获取一个需要的`RiskAssessor` bean。
```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...
}
```
**注** 注意使用构建器注入允许`riskAssessor`字段被标记为`final`，这意味着`riskAssessor`后续是不能改变的。

### 18. 使用@SpringBootApplication注解

很多Spring Boot开发者经常使用`@Configuration`，`@EnableAutoConfiguration`，`@ComponentScan`注解他们的main类，由于这些注解如此频繁地一块使用（特别是遵循以上[最佳实践](14. Structuring your code.md)的时候），Spring Boot就提供了一个方便的`@SpringBootApplication`注解作为代替。

`@SpringBootApplication`注解等价于以默认属性使用`@Configuration`，`@EnableAutoConfiguration`和`@ComponentScan`：
```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
**注** `@SpringBootApplication`注解也提供了用于自定义`@EnableAutoConfiguration`和`@ComponentScan`属性的别名（aliases）。

### 19. 运行应用程序

将应用打包成jar，并使用内嵌HTTP服务器的一个最大好处是，你可以像其他方式那样运行你的应用程序。调试Spring Boot应用也很简单，你都不需要任何特殊IDE插件或扩展！

**注**：本章节只覆盖基于jar的打包，如果选择将应用打包成war文件，你最好参考相关的服务器和IDE文档。

### 19.1. 从IDE中运行

你可以从IDE中运行Spring Boot应用，就像一个简单的Java应用，但首先需要导入项目。导入步骤取决于你的IDE和构建系统，大多数IDEs能够直接导入Maven项目，例如Eclipse用户可以选择`File`菜单的`Import…​` --> `Existing Maven Projects`。

如果不能直接将项目导入IDE，你可以使用构建系统生成IDE的元数据。Maven有针对[Eclipse](http://maven.apache.org/plugins/maven-eclipse-plugin/)和[IDEA](http://maven.apache.org/plugins/maven-idea-plugin/)的插件；Gradle为[各种IDEs](http://www.gradle.org/docs/current/userguide/ide_support.html)提供插件。

**注** 如果意外地多次运行一个web应用，你将看到一个"端口已被占用"的错误。STS用户可以使用`Relaunch`而不是`Run`按钮，以确保任何存在的实例是关闭的。

### 19.2. 作为一个打包后的应用运行

如果使用Spring Boot Maven或Gradle插件创建一个可执行jar，你可以使用`java -jar`运行应用。例如：
```shell
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar
```
Spring Boot支持以远程调试模式运行一个打包的应用，下面的命令可以为应用关联一个调试器：
```shell
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myproject-0.0.1-SNAPSHOT.jar
```

### 19.3. 使用Maven插件运行

Spring Boot Maven插件包含一个`run`目标，可用来快速编译和运行应用程序，并且跟在IDE运行一样支持热加载。
```shell
$ mvn spring-boot:run
```
你可以使用一些有用的操作系统环境变量：
```shell
$ export MAVEN_OPTS=-Xmx1024m -XX:MaxPermSize=128M
```

### 19.4. 使用Gradle插件运行

Spring Boot Gradle插件也包含一个`bootRun`任务，可用来运行你的应用程序。无论你何时import `spring-boot-gradle-plugin`，`bootRun`任务总会被添加进去。
```shell
$ gradle bootRun
```
你可能想使用一些有用的操作系统环境变量：
```shell
$ export JAVA_OPTS=-Xmx1024m -XX:MaxPermSize=128M
```

### 19.5. 热交换

由于Spring Boot应用只是普通的Java应用，所以JVM热交换（hot-swapping）也能开箱即用。不过JVM热交换能替换的字节码有限制，想要更彻底的解决方案可以使用[Spring Loaded](https://github.com/spring-projects/spring-loaded)项目或[JRebel](http://zeroturnaround.com/software/jrebel/)。`spring-boot-devtools`模块也支持应用快速重启(restart)。

详情参考下面的[Chapter 20, Developer tools](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-devtools)和[“How-to”](../IX. ‘How-to’ guides/README.md)章节。

###    开发者工具
Spring Boot包含了一些额外的工具集，用于提升Spring Boot应用的开发体验。`spring-boot-devtools`模块可以included到任何模块中，以提供development-time特性，你只需简单的将该模块的依赖添加到构建中：

**Maven**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```
**Gradle**
```properties
dependencies {
    compile("org.springframework.boot:spring-boot-devtools")
}
```
**注** 在运行一个完整的，打包过的应用时，开发者工具（devtools）会被自动禁用。如果应用使用`java -jar`或特殊的类加载器启动，都会被认为是一个产品级的应用（production application），从而禁用开发者工具。为了防止devtools传递到项目中的其他模块，设置该依赖级别为optional是个不错的实践。不过Gradle不支持`optional`依赖，所以你可能要了解下[propdeps-plugin](https://github.com/spring-projects/gradle-plugins/tree/master/propdeps-plugin)。如果想确保devtools绝对不会包含在一个产品级构建中，你可以使用`excludeDevtools`构建属性彻底移除该JAR，Maven和Gradle都支持该属性。

###     默认属性

Spring Boot支持的一些库（libraries）使用缓存提高性能，比如Thymeleaf将缓存模板以避免重复解析XML源文件。虽然缓存在生产环境很有用，但开发期间就是个累赘了。如果在IDE里修改了模板，你可能会想立即看到结果。

缓存选项通常配置在`application.properties`文件中，比如Thymeleaf提供了`spring.thymeleaf.cache`属性，`spring-boot-devtools`模块会自动应用敏感的`development-time`配置，而不是手动设置这些属性。

**注** 查看[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)获取完整的被应用的属性列表。

###     自动重启

如果应用使用`spring-boot-devtools`，则只要classpath下的文件有变动，它就会自动重启。这在使用IDE时非常有用，因为可以很快得到代码改变的反馈。默认情况下，classpath下任何指向文件夹的实体都会被监控，注意一些资源的修改比如静态assets，视图模板不需要重启应用。

**触发重启** 由于DevTools监控classpath下的资源，所以唯一触发重启的方式就是更新classpath。引起classpath更新的方式依赖于你使用的IDE，在Eclipse里，保存一个修改的文件将引起classpath更新，并触发重启。在IntelliJ IDEA中，构建工程（Build → Make Project）有同样效果。

**注** 你也可以通过支持的构建工具（比如，Maven和Gradle）启动应用，只要开启fork功能，因为DevTools需要一个隔离的应用类加载器执行正确的操作。Gradle默认支持该行为，按照以下配置可强制Maven插件fork进程：
```properties
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```
自动重启跟LiveReload可以一起很好的工作，具体参考[下面章节](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-devtools-livereload)。如果你使用JRebel，自动重启将禁用以支持动态类加载，其他devtools特性，比如LiveReload，属性覆盖仍旧可以使用。

DevTools依赖应用上下文的shutdown钩子来关闭处于重启过程的应用，如果禁用shutdown钩子（`SpringApplication.setRegisterShutdownHook(false)`），它将不能正常工作。

当判定classpath下实体的改变是否会触发重启时，DevTools自动忽略以下工程：`spring-boot`，`spring-boot-devtools`，`spring-boot-autoconfigure`，`spring-boot-actuator`和`spring-boot-starter`。

**Restart vs Reload** Spring Boot提供的重启技术是通过使用两个类加载器实现的。没有变化的类（比如那些第三方jars）会加载进一个基础（basic）classloader，正在开发的类会加载进一个重启（restart）classloader。当应用重启时，restart类加载器会被丢弃，并创建一个新的。这种方式意味着应用重启通常比冷启动（cold starts）快很多，因为基础类加载器已经可用，并且populated（意思是基础类加载器加载的类比较多？）。

如果发现重启对于你的应用来说不够快，或遇到类加载的问题，那你可以考虑reload技术，比如[JRebel](http://zeroturnaround.com/software/jrebel/)，这些技术是通过重写它们加载过的类实现的。[Spring Loaded](https://github.com/spring-projects/spring-loaded)提供了另一种选择，然而很多框架不支持它，也得不到商业支持。


###       排除资源

某些资源的变化没必要触发重启，比如Thymeleaf模板可以随时编辑。默认情况下，位于`/META-INF/maven`，`/META-INF/resources`，`/resources`，`/static`，`/public`或`/templates`下的资源变更不会触发重启，但会触发实时加载（live reload）。你可以使用`spring.devtools.restart.exclude`属性自定义这些排除规则，比如，为了只排除`/static`和`/public`，你可以这样设置：
```properties
spring.devtools.restart.exclude=static/**,public/**
```

**注** 如果你想保留默认属性，并添加其他的排除规则，可以使用`spring.devtools.restart.additional-exclude`属性作为代替。

###       查看其他路径

如果想让应用在改变没有位于classpath下的文件时也会重启或重新加载，你可以使用`spring.devtools.restart.additional-paths`属性来配置监控变化的额外路径。你可以使用[上面描述](./20.2.1 Excluding resources.md)过的`spring.devtools.restart.exclude`属性去控制额外路径下的变化是否触发一个完整重启或只是一个实时重新加载。

###       禁用重启

如果不想使用重启特性，你可以通过`spring.devtools.restart.enabled`属性来禁用它，通常情况下可以在`application.properties`文件中设置（依旧会初始化重启类加载器，但它不会监控文件变化）。

如果需要彻底禁用重启支持，比如，不能跟某个特殊库一块工作，你需要在调用`SpringApplication.run(…​)`之前设置一个系统属性，如下：
```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

###       使用触发器文件

如果使用一个IDE连续不断地编译变化的文件，你可能倾向于只在特定时间触发重启，触发器文件可以帮你实现该功能。触发器文件是一个特殊的文件，只有修改它才能实际触发一个重启检测。改变该文件只会触发检测，实际的重启只会在Devtools发现它必须这样做的时候，触发器文件可以手动更新，也可以通过IDE插件更新。

使用`spring.devtools.restart.trigger-file`属性可以指定触发器文件。

**注** 你可能想将`spring.devtools.restart.trigger-file`属性设置为[全局设置](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-devtools-globalsettings)，这样所有的工程表现都会相同。

###       自定义restart类加载器

正如以上[Restart vs Reload](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-spring-boot-restart-vs-reload)章节讨论的，重启功能是通过两个类加载器实现的。对于大部分应用来说是没问题的，但有时候它可能导致类加载问题。

默认情况，在IDE里打开的项目会通过'restart'类加载器加载，其他常规的`.jar`文件会使用'basic'类加载器加载。如果你工作在一个多模块的项目下，并且不是每个模块都导入IDE里，你可能需要自定义一些东西。你需要创建一个`META-INF/spring-devtools.properties`文件，`spring-devtools.properties`文件可以包含`restart.exclude.`，`restart.include.`前缀的属性。`include`元素定义了那些需要加载进'restart'类加载器中的实体，`exclude`元素定义了那些需要加载进'basic'类加载器中的实体，这些属性的值是一个将应用到classpath的正则表达式。

例如：
```properties
restart.include.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```
**注** 所有属性的keys必须唯一，只要以`restart.include.`或`restart.exclude.`开头都会考虑进去。所有来自classpath的`META-INF/spring-devtools.properties`都会被加载，你可以将文件打包进工程或工程使用的库里。

###       已知限制

重启功能不能跟使用标准`ObjectInputStream`反序列化的对象工作，如果需要反序列化数据，你可能需要使用Spring的`ConfigurableObjectInputStream`，并结合`Thread.currentThread().getContextClassLoader()`。

不幸的是，一些第三方库反序列化时没有考虑上下文类加载器，如果发现这样的问题，你需要请求原作者给处理下。

###     LiveReload

`spring-boot-devtools`模块包含一个内嵌的LiveReload服务器，它可以在资源改变时触发浏览器刷新。LiveReload浏览器扩展可以免费从[livereload.com](http://livereload.com/extensions/)站点获取，支持Chrome，Firefox，Safari等浏览器。

如果不想在运行应用时启动LiveReload服务器，你可以将`spring.devtools.livereload.enabled`属性设置为false。

**注** 每次只能运行一个LiveReload服务器，如果你在IDE中启动多个应用，只有第一个能够获得动态加载功能。

###     全局设置

在`$HOME`文件夹下添加一个`.spring-boot-devtools.properties`的文件可以用来配置全局的devtools设置（注意文件名以"."开头），添加进该文件的任何属性都会应用到你机器上使用该devtools的Spring Boot应用。例如，想使用触发器文件进行重启，可以添加如下配置：

~/.spring-boot-devtools.properties.
```properties
spring.devtools.reload.trigger-file=.reloadtrigger
```

###     远程应用

Spring Boot开发者工具并不仅限于本地开发，在运行远程应用时你也可以使用一些特性。远程支持是可选的，通过设置`spring.devtools.remote.secret`属性可以启用它，例如：
```properties
spring.devtools.remote.secret=mysecret
```
**注** 在远程应用上启用`spring-boot-devtools`有一定的安全风险，生产环境中最好不要使用。

远程devtools支持分两部分：一个是接收连接的服务端端点，另一个是运行在IDE里的客户端应用。如果设置`spring.devtools.remote.secret`属性，服务端组件会自动启用，客户端组件必须手动启动。

###       运行远程客户端应用
远程客户端应用程序（remote client application）需要在IDE中运行，你需要使用跟将要连接的远程应用相同的classpath运行`org.springframework.boot.devtools.RemoteSpringApplication`，传参为你要连接的远程应用URL。例如，你正在使用Eclipse或STS，并有一个部署到Cloud Foundry的`my-app`工程，远程连接该应用需要做以下操作：
* 从`Run`菜单选择`Run Configurations…`。
* 创建一个新的`Java Application`启动配置（launch configuration）。
* 浏览`my-app`工程。
* 将`org.springframework.boot.devtools.RemoteSpringApplication`作为main类。
* 将`https://myapp.cfapps.io`作为参数传递给`RemoteSpringApplication`（或其他任何远程URL）。

运行中的远程客户端看起来如下：
```shell
 .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote :: 1.4.1.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```

**注** 因为远程客户端使用的classpath跟真实应用相同，所以它能直接读取应用配置，这就是`spring.devtools.remote.secret`如何被读取和传递给服务器做验证的。

强烈建议使用`https://`作为连接协议，这样传输通道是加密的，密码也不会被截获。

如果需要使用代理连接远程应用，你需要配置`spring.devtools.remote.proxy.host`和`spring.devtools.remote.proxy.port`属性。


###       远程更新
远程客户端将监听应用的classpath变化，任何更新的资源都会发布到远程应用，并触发重启，这在你使用云服务迭代某个特性时非常有用。通常远程更新和重启比完整rebuild和deploy快多了。

**注** 文件只有在远程客户端运行时才监控。如果你在启动远程客户端之前改变一个文件，它是不会被发布到远程server的。

###       远程调试通道
Java的远程调试在诊断远程应用问题时很有用，不幸的是，当应用部署在你的数据中心外时，它并不总能够启用远程调试。如果你使用基于容器的技术，比如Docker，远程调试设置起来非常麻烦。

为了突破这些限制，devtools支持基于HTTP的远程调试通道。远程客户端在8000端口提供一个本地server，这样远程debugger就可以连接了。一旦连接建立，调试信息就通过HTTP发送到远程应用。你可以使用`spring.devtools.remote.debug.local-port`属性设置不同的端口。

你需要确保远程应用启动时开启了远程调试功能，通常，这可以通过配置`JAVA_OPTS`实现，例如，对于Cloud Foundry，你可以将以下内容添加到manifest.yml：
```properties
---
  env:
    JAVA_OPTS: "-Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n"
```
**注** 注意你不需要传递一个`address=NNNN`的配置项到`-Xrunjdwp`，如果遗漏了，java会使用一个随机可用端口。

调试基于Internet的远程服务可能很慢，你可能需要增加IDE的超时时间。例如，在Eclipse中你可以从`Preferences…`选择`Java` -> `Debug`，改变`Debugger timeout (ms)`为更合适的值（60000在多数情况下就能解决）。

### 21. 打包用于生产的应用

可执行jars可用于生产部署。由于它们是自包含的，非常适合基于云的部署。关于其他“生产准备”的特性，比如健康监控，审计和指标REST，或JMX端点，可以考虑添加`spring-boot-actuator`。具体参考[Part V, “Spring Boot Actuator: Production-ready features”](../V. Spring Boot Actuator/README.md)。

### 22. 接下来阅读什么
现在你应该明白怎么结合最佳实践使用Spring Boot，接下来可以深入学习特殊的部分[Spring Boot features](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features)，或者你可以跳过开头，阅读Spring Boot的[production ready](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#production-ready)部分。
