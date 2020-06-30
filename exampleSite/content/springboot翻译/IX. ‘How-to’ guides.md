---
title: IX. ‘How-to’ guides
date: 2018-06-15 18:16:40
tags: ["springboot"]
categories: ["springboot翻译"]
---
### 68. Spring Boot应用

###     创建自己的FailureAnalyzer
[FailureAnalyzer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/diagnostics/FailureAnalyzer.html)是拦截启动时的异常并将它转换为可读消息的很好方式，Spring Boot为应用上下文相关异常， JSR-303校验等提供分析器，实际上创建你自己的分析器也相当简单。

`AbstractFailureAnalyzer`是`FailureAnalyzer`的一个方便扩展，根据指定类型的异常是否出现来进行处理。你可以继承它，这样就可以处理实际出现的异常。如果出于某些原因，不能处理该异常，那就返回`null`让其他实现处理。

`FailureAnalyzer`的实现需要注册到`META-INF/spring.factories`，以下注册了`ProjectConstraintViolationFailureAnalyzer`：
```properties
org.springframework.boot.diagnostics.FailureAnalyzer=\
com.example.ProjectConstraintViolationFailureAnalyzer
```
<!--more-->
### 68.2 解决自动配置问题

Spring Boot自动配置总是尝试尽最大努力去做正确的事，但有时候会失败并且很难说出失败原因。

在每个Spring Boot `ApplicationContext`中都存在一个相当有用的`ConditionEvaluationReport`。如果开启`DEBUG`日志输出，你将会看到它。如果你使用`spring-boot-actuator`，则会有一个`autoconfig`的端点，它将以JSON形式渲染该报告。你还可以使用它调试应用程序，并能查看Spring Boot运行时都添加了哪些特性（及哪些没添加）。

通过查看源码和javadoc可以获取更多问题的答案，以下是一些经验：

* 查找名为`*AutoConfiguration`的类并阅读源码，特别是`@Conditional*`注解，这可以帮你找出它们启用哪些特性及何时启用。
将`--debug`添加到命令行或添加系统属性`-Ddebug`可以在控制台查看日志，该日志会记录你的应用中所有自动配置的决策。在运行Actuator的app中，通过查看`autoconfig`端点（`/autoconfig`或等效的JMX）可以获取相同信息。
* 查找`@ConfigurationProperties`的类（比如[ServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)）并看下有哪些可用的外部配置选项。`@ConfigurationProperties`类有一个用于充当外部配置前缀的`name`属性，因此`ServerProperties`的`prefix="server"`，它的配置属性有`server.port`，`server.address`等。在运行Actuator的应用中可以查看`configprops`端点。
* 查看`RelaxedPropertyResolver`明确地将配置从`Environment`暴露出去，它经常会使用前缀。
* 查看`@Value`注解，它直接绑定到`Environment`。相比`RelaxedPropertyResolver`，这种方式稍微缺乏灵活性，但它也允许松散的绑定，特别是OS环境变量（所以`CAPITALS_AND_UNDERSCORES`是`period.separated`的同义词）。
* 查看`@ConditionalOnExpression`注解，它根据SpEL表达式的结果来开启或关闭特性，通常使用解析自`Environment`的占位符进行计算。


### 68.3 启动前自定义Environment或ApplicationContext

每个`SpringApplication`都有`ApplicationListeners`和`ApplicationContextInitializers`，用于自定义上下文（context）或环境(environment)。Spring Boot从`META-INF/spring.factories`下加载很多这样的内部使用的自定义，有很多方法可以注册其他的自定义：

* 以编程方式为每个应用注册自定义，通过在`SpringApplication`运行前调用它的`addListeners`和`addInitializers`方法来实现。
* 以声明方式为每个应用注册自定义，通过设置`context.initializer.classes`或`context.listener.classes`来实现。
* 以声明方式为所有应用注册自定义，通过添加一个`META-INF/spring.factories`并打包成一个jar文件（该应用将它作为一个库）来实现。

`SpringApplication`会给监听器（即使是在上下文被创建之前就存在的）发送一些特定的`ApplicationEvents`，然后也会注册监听`ApplicationContext`发布的事件的监听器，查看Spring Boot特性章节中的[Section 23.5, “Application events and listeners” ](../IV. Spring Boot features/23.5. Application events and listeners.md)可以获取完整列表。

在应用上下文刷新前使用`EnvironmentPostProcessor`自定义`Environment`是可能的，每个实现都需要注册到`META-INF/spring.factories`：
```properties
org.springframework.boot.env.EnvironmentPostProcessor=com.example.YourEnvironmentPostProcessor
```

### 68.4 构建ApplicationContext层次结构（添加父或根上下文）

你可以使用`ApplicationBuilder`类创建parent/child `ApplicationContext`层次结构，查看'Spring Boot特性'章节的[Section 23.4, “Fluent builder API” ](../IV. Spring Boot features/23.4. Fluent builder API.md)获取更多信息。

### 68.5 创建no-web应用

不是所有的Spring应用都必须是web应用（或web服务）。如果你想在`main`方法中执行一些代码，但需要启动一个Spring应用去设置需要的底层设施，那使用Spring Boot的`SpringApplication`特性可以很容易实现。`SpringApplication`会根据它是否需要一个web应用来改变它的`ApplicationContext`类，首先你需要做的是去掉servlet API依赖，如果不能这样做（比如基于相同的代码运行两个应用），那你可以明确地调用`SpringApplication.setWebEnvironment(false)`或设置`applicationContextClass`属性（通过Java API或使用外部配置）。你想运行的，作为业务逻辑的应用代码可以实现为一个`CommandLineRunner`，并将上下文降级为一个`@Bean`定义。

### 69. 属性&配置

###      运行时暴露属性

相对于在项目构建配置中硬编码某些配置，你可以使用已存在的构建配置自动暴露它们，Maven和Gradle都支持。

###        使用Maven自动暴露属性

你可以使用Maven的资源过滤（resource filter）自动暴露来自Maven项目的属性，如果使用`spring-boot-starter-parent`，你可以通过`@..@`占位符引用Maven项目的属性，例如：
```properties
app.encoding=@project.build.sourceEncoding@
app.java.version=@java.version@
```
**注** 如果启用`addResources`标识，`spring-boot:run`可以将`src/main/resources`直接添加到classpath（出于热加载目的），这就绕过了资源过滤和本特性。你可以使用`exec:java`目标进行替代，或自定义该插件的配置，具体查看[插件使用页面](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/usage.html)。

如果不使用starter parent，你需要将以下片段添加到`pom.xml`中（`<build/>`元素内）：
```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```
和（`<plugins/>`元素内）：
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <configuration>
        <delimiters>
            <delimiter>@</delimiter>
        </delimiters>
        <useDefaultDelimiters>false</useDefaultDelimiters>
    </configuration>
</plugin>
```

**注** 如果你在配置中使用标准的Spring占位符（比如`${foo}`）且没有将`useDefaultDelimiters`属性设置为`false`，那构建时这些属性将被暴露出去。

###        使用Gradle自动暴露属性

你可以通过配置Java插件的`processResources`任务自动暴露来自Gradle项目的属性：
```gradle
processResources {
    expand(project.properties)
}
```
然后你可以通过占位符引用Gradle项目的属性：
```properties
app.name=${name}
app.description=${description}
```

**注** Gradle的`expand`方法使用Groovy的`SimpleTemplateEngine`转换`${..}`占位符，`${..}`这种格式跟Spring自身的属性占位符机制冲突，想要自动暴露Spring属性占位符，你需要将其进行编码，比如`\${..}`。

### 69.2. 外部化SpringApplication配置

SpringApplication已经被属性化（主要是setters），所以你可以在创建应用时使用它的Java API修改其行为，或者使用以`spring.main.*`为key的属性来外部化这些配置。比如，在`application.properties`中可能会有以下内容：
```java
spring.main.web-environment=false
spring.main.banner-mode=off
```
这样，Spring Boot在启动时将不会显示banner，并且该应用也不是一个web应用。

**注** 以上示例也展示在属性名中使用下划线（`_`）和中划线（`-`）的灵活绑定。

外部配置定义的属性会覆盖创建`ApplicationContext`时通过Java API指定的值，让我们看如下应用：
```java
new SpringApplicationBuilder()
    .bannerMode(Banner.Mode.OFF)
    .sources(demo.MyApp.class)
    .run(args);
```
并使用以下配置：
```properties
spring.main.sources=com.acme.Config,com.acme.ExtraConfig
spring.main.banner-mode=console
```
实际的应用将显示banner（被配置覆盖），并为`ApplicationContext`指定3个sources，依次为：`demo.MyApp`，`com.acme.Config`，`com.acme.ExtraConfig`。


### 69.3 改变应用程序外部配置文件的位置

默认情况下，来自不同源的属性以一个定义好的顺序添加到Spring的`Environment`中（精确顺序可查看'Sprin Boot特性'章节的[Chapter 24, Externalized Configuration](../IV. Spring Boot features/24. Externalized Configuration.md)）。

为应用程序源添加`@PropertySource`注解是一种很好的添加和修改源顺序的方法。传递给`SpringApplication`静态便利设施（convenience）方法的类和使用`setSources()`添加的类都会被检查，以查看它们是否有`@PropertySources`，如果有，这些属性会被尽可能早的添加到`Environment`里，以确保`ApplicationContext`生命周期的所有阶段都能使用。以这种方式添加的属性优先级低于任何使用默认位置（比如`application.properties`）添加的属性，系统属性，环境变量或命令行参数。

你也可以提供系统属性（或环境变量）来改变该行为：

* `spring.config.name`（`SPRING_CONFIG_NAME`）是根文件名，默认为`application`。
* `spring.config.location`（`SPRING_CONFIG_LOCATION`）是要加载的文件（例如，一个classpath资源或URL）。Spring Boot为该文档设置一个单独的`Environment`属性，它可以被系统属性，环境变量或命令行参数覆盖。

不管你在environment设置什么，Spring Boot都将加载上面讨论过的`application.properties`。如果使用YAML，那具有`.yml`扩展的文件默认也会被添加到该列表，详情参考[ConfigFileApplicationListener](http://github.com/spring-projects/spring-boot/tree/master/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java)

### 69.4 使用'short'命令行参数

有些人喜欢使用（例如）`--port=9000`代替`--server.port=9000`来设置命令行配置属性。你可以通过在`application.properties`中使用占位符来启用该功能，比如：
```properties
server.port=${port:8080}
```
**注** 如果你继承自`spring-boot-starter-parent` POM，为了防止和Spring格式的占位符产生冲突，`maven-resources-plugins`默认的过滤令牌（filter token）已经从`${*}`变为`@`（即`@maven.token@`代替`${maven.token}`）。如果直接启用maven对`application.properties`的过滤，你可能想使用[其他的分隔符](http://maven.apache.org/plugins/maven-resources-plugin/resources-mojo.html#delimiters)替换默认的过滤令牌。

**注** 在这种特殊的情况下，端口绑定能够在一个PaaS环境下工作，比如Heroku和Cloud Foundry，因为在这两个平台中`PORT`环境变量是自动设置的，并且Spring能够绑定`Environment`属性的大写同义词。


###     使用YAML配置外部属性

YAML是JSON的一个超集，可以非常方便的将外部配置以层次结构形式存储起来，比如：
```json
spring:
    application:
        name: cruncher
    datasource:
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost/test
server:
    port: 9000
```
创建一个`application.yml`文件，将它放到classpath的根目录下，并添加`snakeyaml`依赖（Maven坐标为`org.yaml:snakeyaml`，如果你使用`spring-boot-starter`那就已经包含了）。一个YAML文件会被解析为一个Java `Map<String,Object>`（和一个JSON对象类似），Spring Boot会平伸该map，这样它就只有1级深度，并且有period-separated的keys，跟人们在Java中经常使用的`Properties`文件非常类似。
上面的YAML示例对应于下面的`application.properties`文件：
```java
spring.application.name=cruncher
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost/test
server.port=9000
```
查看'Spring Boot特性'章节的[Section 24.6, “Using YAML instead of Properties”](../IV. Spring Boot features/24.6. Using YAML instead of Properties.md)可以获取更多关于YAML的信息。

### 69.6 设置生效的Spring profiles

Spring `Environment`有一个API可以设置生效的profiles，但通常你会通过系统属性（`spring.profiles.active`）或OS环境变量（`SPRING_PROFILES_ACTIVE`）设置。比如，使用一个`-D`参数启动应用程序（记着把它放到`main`类或jar文件之前）：
```shell
$ java -jar -Dspring.profiles.active=production demo-0.0.1-SNAPSHOT.jar
```
在Spring Boot中，你也可以在`application.properties`里设置生效的profile，例如：
```java
spring.profiles.active=production
```
通过这种方式设置的值会被系统属性或环境变量替换，但不会被`SpringApplicationBuilder.profiles()`方法替换。因此，后面的Java API可用来在不改变默认设置的情况下增加profiles。

想要获取更多信息可查看'Spring Boot特性'章节的[Chapter 25, Profiles](..//IV. Spring Boot features/25. Profiles.md)。

### 69.7 根据环境改变配置

一个YAML文件实际上是一系列以`---`线分割的文档，每个文档都被单独解析为一个平坦的（flattened）map。

如果一个YAML文档包含一个`spring.profiles`关键字，那profiles的值（以逗号分割的profiles列表）将被传入Spring的`Environment.acceptsProfiles()`方法，并且如果这些profiles的任何一个被激活，对应的文档被包含到最终的合并中（否则不会）。

示例：
```json
server:
    port: 9000
---

spring:
    profiles: development
server:
    port: 9001

---

spring:
    profiles: production
server:
    port: 0
```
在这个示例中，默认的端口是`9000`，但如果Spring profile `development`生效则该端口是`9001`，如果`production`生效则它是`0`。

YAML文档以它们出现的顺序合并，所以后面的值会覆盖前面的值。

想要使用profiles文件完成同样的操作，你可以使用`application-${profile}.properties`指定特殊的，profile相关的值。


### 69.8 发现外部属性的内置选项

Spring Boot在运行时会将来自`application.properties`（或`.yml`）的外部属性绑定到应用，因为不可能将所有支持的属性放到一个地方，classpath下的其他jar也有支持的属性。

每个运行中且有Actuator特性的应用都会有一个`configprops`端点，它能够展示所有边界和可通过`@ConfigurationProperties`绑定的属性。

附录中包含一个[application.properties](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#common-application-properties)示例，它列举了Spring Boot支持的大多数常用属性，查看`@ConfigurationProperties`，`@Value`，还有不经常使用的`RelaxedEnvironment`的源码可获取最权威的属性列表。

### 70.  内嵌servlet容器

### 70.1 为应用添加Servlet，Filter或Listener

这里有两种方式可以为应用添加`Servlet`，`Filter`，`ServletContextListener`和其他Servlet支持的特定listeners。你既可以为它们提供Spring beans，也可以为Servlet组件启用扫描（package scan）。

###       使用Spring bean添加Servlet, Filter或Listener

想要添加`Servlet`，`Filter`或Servlet`*Listener`，你只需要为它提供一个`@Bean`定义，这种方式很适合注入配置或依赖。不过，需要注意的是它们不会导致其他很多beans的热初始化，因为它们需要在应用生命周期的早期进行安装（让它依赖`DataSource`或JPA配置不是好主意），你可以通过懒加载突破该限制（在第一次使用时才初始化）。

对于`Filters`或`Servlets`，你可以通过`FilterRegistrationBean`或`ServletRegistrationBean`添加映射和初始化参数。

**注** 在一个filter注册时，如果没指定`dispatcherType`，它将匹配`FORWARD`，`INCLUDE`和`REQUEST`。如果启用异步，它也将匹配`ASYNC`。如果迁移`web.xml`中没有`dispatcher`元素的filter，你需要自己指定一个`dispatcherType`：
```java
@Bean
public FilterRegistrationBean myFilterRegistration() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setDispatcherTypes(DispatcherType.REQUEST);
    ....

    return registration;
}
```

**禁止Servlet或Filter的注册**

如上所述，任何`Servlet`或`Filter` beans都将自动注册到servlet容器。不过，为特定的`Filter`或`Servlet` bean创建一个registration，并将它标记为disabled，可以禁用该filter或servlet。例如：
```java
@Bean
public FilterRegistrationBean registration(MyFilter filter) {
    FilterRegistrationBean registration = new FilterRegistrationBean(filter);
    registration.setEnabled(false);
    return registration;
}
```

###       使用classpath扫描添加Servlets, Filters和Listeners

通过把`@ServletComponentScan`注解到一个`@Configuration`类并指定包含要注册组件的package(s)，可以将`@WebServlet`，`@WebFilter`和`@WebListener`注解的类自动注册到内嵌servlet容器。默认情况下，`@ServletComponentScan`将从被注解类的package开始扫描。

###      使用Tomcat的LegacyCookieProcessor

Spring Boot使用的内嵌Tomcat不能开箱即用的支持`Version 0`的Cookie格式，你可能会看到以下错误：
```java
java.lang.IllegalArgumentException: An invalid character [32] was present in the Cookie value
```
可以的话，你需要考虑将代码升级到只存储遵从最新版Cookie定义的值。如果不能改变写入的cookie，你可以配置Tomcat使用`LegacyCookieProcessor`。通过向`EmbeddedServletContainerCustomizer` bean添加一个`TomcatContextCustomizer`可以开启`LegacyCookieProcessor`：
```java
@Bean
public EmbeddedServletContainerCustomizer cookieProcessorCustomizer() {
    return new EmbeddedServletContainerCustomizer() {

        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            if (container instanceof TomcatEmbeddedServletContainerFactory) {
                ((TomcatEmbeddedServletContainerFactory) container)
                        .addContextCustomizers(new TomcatContextCustomizer() {

                    @Override
                    public void customize(Context context) {
                        context.setCookieProcessor(new LegacyCookieProcessor());
                    }

                });
            }
        }

    };
}
```

### 70.11 使用Jetty替代Tomcat

Spring Boot starters（特别是`spring-boot-starter-web`）默认都使用Tomcat作为内嵌容器。想使用Jetty替代Tomcat，你需要排除那些Tomcat的依赖并包含Jetty的依赖。为了简化这种事情的处理，Spring Boot将Tomcat和Jetty的依赖捆绑在一起，然后提供了单独的starters。

Maven示例：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```
Gradle示例：
```gradle
configurations {
    compile.exclude module: "spring-boot-starter-tomcat"
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web:1.4.1.RELEASE")
    compile("org.springframework.boot:spring-boot-starter-jetty:1.4.1.RELEASE")
    // ...
}
```


### 70.12 配置Jetty

通常你可以遵循[Section 69.8, “Discover built-in options for external properties”](./69.8 Discover built-in options for external properties.md)关于`@ConfigurationProperties`（此处主要是`ServerProperties`）的建议，但也要看下`EmbeddedServletContainerCustomizer`。

Jetty API相当丰富，一旦获取到`JettyEmbeddedServletContainerFactory`，你就可以使用很多方式修改它，或更彻底地就是添加你自己的`JettyEmbeddedServletContainerFactory`。

### 70.13 使用Undertow替代Tomcat

使用Undertow替代Tomcat和[使用Jetty替代Tomcat](./70.11 Use Jetty instead of Tomcat.md)非常类似。你需要排除Tomat依赖，并包含Undertow starter。

Maven示例：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```
Gradle示例：
```gradle
configurations {
    compile.exclude module: "spring-boot-starter-tomcat"
}

dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web:1.3.0.BUILD-SNAPSHOT")
    compile 'org.springframework.boot:spring-boot-starter-undertow:1.3.0.BUILD-SNAPSHOT")
    // ...
}

```

### 70.14 配置Undertow

通常你可以遵循[Section 69.8, “Discover built-in options for external properties”](./69.8 Discover built-in options for external properties.md)关于`@ConfigurationProperties`（此处主要是`ServerProperties`和`ServerProperties.Undertow`），但也要看下`EmbeddedServletContainerCustomizer`。

一旦获取到`UndertowEmbeddedServletContainerFactory`，你就可以使用`UndertowBuilderCustomizer`修改Undertow的配置以满足你的需求，或更彻底地就是添加你自己的`UndertowEmbeddedServletContainerFactory`。

### 70.15 启用Undertow的多监听器

将`UndertowBuilderCustomizer`添加到`UndertowEmbeddedServletContainerFactory`，然后使用`Builder`添加一个listener：
```java
@Bean
public UndertowEmbeddedServletContainerFactory embeddedServletContainerFactory() {
    UndertowEmbeddedServletContainerFactory factory = new UndertowEmbeddedServletContainerFactory();
    factory.addBuilderCustomizers(new UndertowBuilderCustomizer() {

        @Override
        public void customize(Builder builder) {
            builder.addHttpListener(8080, "0.0.0.0");
        }

    });
    return factory;
}
```

### 70.16 使用Tomcat 7.x或8.0

Spring Boot可以使用Tomcat7&8.0，但默认使用的是Tomcat8.5。如果不能使用Tomcat8.5（例如，因为你使用的是Java1.6），你需要改变classpath去引用一个不同版本。

### 70.16.1 通过Maven使用Tomcat 7.x或8.0

如果正在使用starters 和parent，你只需要改变Tomcat的`version`属性，并添加`tomcat-juli`依赖。比如，对于一个简单的webapp或service：
```xml
<properties>
    <tomcat.version>7.0.59</tomcat.version>
</properties>
<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat</groupId>
        <artifactId>tomcat-juli</artifactId>
        <version>${tomcat.version}</version>
    </dependency>
    ...
</dependencies>
```

### 70.16.2 通过Gradle使用Tomcat7.x或8.0

对于Gradle，你可以通过设置`tomcat.version`属性改变Tomcat的版本，然后添加`tomcat-juli`依赖：
```gradle
ext['tomcat.version'] = '7.0.59'
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web'
    compile group:'org.apache.tomcat', name:'tomcat-juli', version:property('tomcat.version')
}
```

### 70.17 使用Jetty9.2

Spring Boot可以使用Jetty9.2，但默认使用的是Jetty9.3。如果不能使用Jetty9.3（例如，因为你使用的是Java7），你需要改变classpath去引用Jetty9.2。

### 70.17.1 通过Maven使用Jetty9.2

如果正在使用starters和parent，你只需添加Jetty starter并覆盖`jetty.version`属性：
```xml
<properties>
    <jetty.version>9.2.17.v20160517</jetty.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
</dependencies>
```

### 70.17.2 通过Gradle使用Jetty 9.2

对于Gradle，你需要设置`jetty.version`属性，例如对于一个简单的webapp或service：
```gradle
ext['jetty.version'] = '9.2.17.v20160517'
dependencies {
    compile ('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    compile ('org.springframework.boot:spring-boot-starter-jetty')
}
```

###      使用Jetty 8

Spring Boot支持Jetty 8，但默认使用的是Jetty 9.3。如果不能使用Jetty 9.3（比如因为你使用的是Java 1.6），你需要改变classpath去引用Jetty 8，还需要排除Jetty的WebSocket相关依赖。

###        通过Maven使用Jetty8

如果正在使用starters和parent，你只需要添加Jetty starter，排除那些需要的WebSocket，并改变version属性。比如，对于一个简单的webapp或service：
```xml
<properties>
    <jetty.version>8.1.15.v20140411</jetty.version>
    <jetty-jsp.version>2.2.0.v201112011158</jetty-jsp.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.eclipse.jetty.websocket</groupId>
                <artifactId>*</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

### 70.18.2 通过Gradle使用Jetty8

你可以设置`jetty.version`属性并排除相关的WebSocket依赖，比如对于一个简单的webapp或service：
```gradle
ext['jetty.version'] = '8.1.15.v20140411'
dependencies {
    compile ('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    compile ('org.springframework.boot:spring-boot-starter-jetty') {
        exclude group: 'org.eclipse.jetty.websocket'
    }
}
```

### 70.19 使用@ServerEndpoint创建WebSocket端点

如果想在使用内嵌容器的Spring Boot应用中使用`@ServerEndpoint`，你需要声明一个单独的`ServerEndpointExporter` `@Bean`：
```java
@Bean
public ServerEndpointExporter serverEndpointExporter() {
    return new ServerEndpointExporter();
}
```
该bean将使用底层的WebSocket容器注册任何被`@ServerEndpoint`注解的beans。当部署到一个单独的servlet容器时，该角色将被一个servlet容器初始化方法执行，`ServerEndpointExporter` bean也就不需要了。

### 70.2 改变HTTP端口

在一个单独的应用中，主HTTP端口默认为`8080`，不过可以使用`server.port`设置（比如，在`application.properties`中或作为系统属性）。由于`Environment`值的宽松绑定，你也可以使用`SERVER_PORT`（比如，作为OS环境变量）。

想要创建`WebApplicationContext`但完全关闭HTTP端点，你可以设置`server.port=-1`（测试时可能有用）。具体详情可查看'Spring Boot特性'章节的[Section 27.3.4, “Customizing embedded servlet containers”](../IV. Spring Boot features/27.3.4 Customizing embedded servlet containers.md)，或[ServerProperties](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)源码。

### 70.20 启用HTTP响应压缩

Jetty，Tomcat和Undertow支持HTTP响应压缩，你可以通过设置`server.compression.enabled`启用它：
```properties
server.compression.enabled=true
```
默认情况下，响应信息长度至少2048字节才能触发压缩，通过`server.compression.min-response-size`属性可以改变该长度。另外，只有响应的content type为以下其中之一时才压缩：

- `text/html`
- `text/xml`
- `text/plain`
- `text/css`

你可以通过`server.compression.mime-types`属性配置。

### 70.3 使用随机未分配的HTTP端口

想扫描获取一个未使用的端口（使用操作系统本地端口以防冲突）可以设置`server.port=0`。

### 70.4 发现运行时的HTTP端口

你可以通过日志输出或它的`EmbeddedServletContainer`的`EmbeddedWebApplicationContext`获取服务器正在运行的端口。获取和确认服务器已经初始化的最好方式是添加一个`ApplicationListener<EmbeddedServletContainerInitializedEvent>`类型的`@Bean`，然后当事件发布时将容器pull出来。

使用`@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`进行测试时，你可以通过`@LocalServerPort`注解将实际端口注入到字段中，例如：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class MyWebIntegrationTests {

    @Autowired
    EmbeddedWebApplicationContext server;

    @LocalServerPort
    int port;

    // ...

}
```
**注** `@LocalServerPort`是`@Value("${local.server.port}")`的元数据，在常规的应用中不要尝试注入端口。正如我们看到的，该值只会在容器初始化后设置。相对于测试，应用代码回调处理的会更早（例如在该值实际可用之前）。

### 70.5 配置SSL

你可以以声明方式配置SSL，一般通过在`application.properties`或`application.yml`设置各种各样的`server.ssl.*`属性，例如：
```json
server.port = 8443
server.ssl.key-store = classpath:keystore.jks
server.ssl.key-store-password = secret
server.ssl.key-password = another-secret
```
查看[Ssl](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/embedded/Ssl.java)获取所有支持的配置。

使用类似于以上示例的配置意味着该应用将不支持端口为8080的普通HTTP连接。Spring Boot不支持通过`application.properties`同时配置HTTP连接器和HTTPS连接器。如果你两个都想要，那就需要以编程的方式配置它们中的一个。推荐使用`application.properties`配置HTTPS，因为HTTP连接器是两个中最容易以编程方式进行配置的，查看[spring-boot-sample-tomcat-multi-connectors](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-samples/spring-boot-sample-tomcat-multi-connectors)可获取示例项目。

###     配置访问日志

通过相应的命令空间可以为Tomcat和Undertow配置访问日志，例如下面是为Tomcat配置的一个[自定义模式](https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Access_Logging)的访问日志：
```properties
server.tomcat.basedir=my-tomcat
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%t %a "%r" %s (%D ms)
```
**注** 日志默认路径为tomcat基础路径下的`logs`目录，该dir默认是个临时目录，所以你可能想改变Tomcat的base目录或为日志指定绝对路径。上述示例中，你可以在相对于应用工作目录的`my-tomcat/logs`访问到日志。

Undertow的访问日志配置方式类似：
```properties
server.undertow.accesslog.enabled=true
server.undertow.accesslog.pattern=%t %a "%r" %s (%D ms)
```
日志存储在相对于应用工作目录的`logs`目录下，可以通过`server.undertow.accesslog.directory`自定义。

###     在前端代理服务器后使用

你的应用可能需要发送`302`跳转或使用指向自己的绝对路径渲染内容。当在代理服务器后面运行时，调用者需要的是代理服务器链接而不是部署应用的实际物理机器地址，通常的解决方式是代理服务器将前端地址放到headers并告诉后端服务器如何拼装链接。

如果代理添加约定的`X-Forwarded-For`和`X-Forwarded-Proto` headers（大多数都是开箱即用的），只要将`application.properties`中的`server.use-forward-headers`设置为`true`，绝对链接就能正确的渲染。

**注** 如果应用运行在Cloud Foundry或Heroku，`server.use-forward-headers`属性没指定的话默认为`true`，其他实例默认为`false`。

###       自定义Tomcat代理配置

如果使用的是Tomcat，你可以配置用于传输"forwarded"信息的headers名：
```properties
server.tomcat.remote-ip-header=x-your-remote-ip-header
server.tomcat.protocol-header=x-your-protocol-header
```
你也可以为Tomcat配置一个默认的正则表达式，用来匹配内部信任的代理。默认情况下，IP地址`10/8`，`192.168/16`，`169.254/16`和`127/8`是被信任的。通过设置`server.tomcat.internal-proxies`属性可以自定义，比如：
```properties
server.tomcat.internal-proxies=192\\.168\\.\\d{1,3}\\.\\d{1,3}
```
**注** 只有在使用配置文件时才需要双反斜线，如果使用YAML，只需要单个反斜线，比如`192\.168\.\d{1,3}\.\d{1,3}`。

**注** 将`internal-proxies`设置为空表示信任所有代理，不要在生产环境使用。

你可以完全控制Tomcat的`RemoteIpValve`配置，只要关掉自动配置（比如设置`server.use-forward-headers=false`）并在`TomcatEmbeddedServletContainerFactory` bean添加一个新value实例。

### 70.8 配置Tomcat

通常你可以遵循[Section 69.8, “Discover built-in options for external properties”](./69.8 Discover built-in options for external properties.md)关于`@ConfigurationProperties`（这里主要的是`ServerProperties`）的建议，但也看下`EmbeddedServletContainerCustomizer`和各种你可以添加的Tomcat-specific的`*Customizers`。

Tomcat APIs相当丰富，一旦获取到`TomcatEmbeddedServletContainerFactory`，你就能够以多种方式修改它，或更彻底地就是添加你自己的`TomcatEmbeddedServletContainerFactory`。

### 70.9 启用Tomcat的多连接器

你可以将`org.apache.catalina.connector.Connector`添加到`TomcatEmbeddedServletContainerFactory`，这就能够允许多连接器，比如HTTP和HTTPS连接器：
```java
@Bean
public EmbeddedServletContainerFactory servletContainer() {
    TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory();
    tomcat.addAdditionalTomcatConnectors(createSslConnector());
    return tomcat;
}

private Connector createSslConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
    try {
        File keystore = new ClassPathResource("keystore").getFile();
        File truststore = new ClassPathResource("keystore").getFile();
        connector.setScheme("https");
        connector.setSecure(true);
        connector.setPort(8443);
        protocol.setSSLEnabled(true);
        protocol.setKeystoreFile(keystore.getAbsolutePath());
        protocol.setKeystorePass("changeit");
        protocol.setTruststoreFile(truststore.getAbsolutePath());
        protocol.setTruststorePass("changeit");
        protocol.setKeyAlias("apitester");
        return connector;
    }
    catch (IOException ex) {
        throw new IllegalStateException("can't access keystore: [" + "keystore"
                + "] or truststore: [" + "keystore" + "]", ex);
    }
}

```

### 71. Spring MVC

### 71.1 编写JSON REST服务

只要添加的有Jackson2依赖，Spring Boot应用中的任何`@RestController`默认都会渲染为JSON响应，例如：
```java
@RestController
public class MyController {

    @RequestMapping("/thing")
    public MyThing thing() {
            return new MyThing();
    }

}
```
只要`MyThing`能够通过Jackson2序列化（比如，一个标准的POJO或Groovy对象），默认[localhost:8080/thing](http://localhost:8080/thing)将响应一个JSON数据。有时在浏览器中你可能看到XML响应，因为浏览器倾向于发送XML accept headers。

###      使用Thymeleaf 3

默认情况下，`spring-boot-starter-thymeleaf`使用的是Thymeleaf 2.1，你可以通过覆盖`thymeleaf.version`和`thymeleaf-layout-dialect.version`属性使用Thymeleaf 3，例如：
```properties
<properties>
    <thymeleaf.version>3.0.0.RELEASE</thymeleaf.version>
    <thymeleaf-layout-dialect.version>2.0.0</thymeleaf-layout-dialect.version>
</dependency>
```
为了避免关于HTML 5模板模式过期，将使用HTML模板模式的警告提醒，你需要显式配置`spring.thymeleaf.mode`为`HTML`，例如：
```properties
spring.thymeleaf.mode: HTML
```
具体操作可查看[Thymeleaf 3示例](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-samples/spring-boot-sample-web-thymeleaf3)。

如果正在使用其他自动配置的Thymeleaf附加组件（Spring Security，Data Attribute或Java 8 Time），你需要使用兼容Thymeleaf 3.0的版本覆盖它们现在的版本。

### 71.2 编写XML REST服务

如果classpath下存在Jackson XML扩展（`jackson-dataformat-xml`），它会被用来渲染XML响应，示例和JSON的非常相似。想要使用它，只需为你的项目添加以下依赖：
```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```
你可能还需要添加Woodstox的依赖，它比JDK提供的默认StAX实现快很多，并且支持良好的格式化输出，提高了namespace处理能力：
```xml
<dependency>
    <groupId>org.codehaus.woodstox</groupId>
    <artifactId>woodstox-core-asl</artifactId>
</dependency>
```
如果Jackson的XML扩展不可用，Spring Boot将使用JAXB（JDK默认提供），不过`MyThing`需要注解`@XmlRootElement`：
```java
@XmlRootElement
public class MyThing {
    private String name;
    // .. getters and setters
}
```
想要服务器渲染XML而不是JSON，你可能需要发送一个`Accept: text/xml`头部（或使用浏览器）。

### 71.3 自定义Jackson ObjectMapper

在一个HTTP交互中，Spring MVC（客户端和服务端）使用`HttpMessageConverters`协商内容转换。如果classpath下存在Jackson，你就获取到`Jackson2ObjectMapperBuilder`提供的默认转换器，这是Spring Boot为你自动配置的实例。

创建的`ObjectMapper`（或用于Jackson XML转换的`XmlMapper`）实例默认有以下自定义属性：

- `MapperFeature.DEFAULT_VIEW_INCLUSION`，默认是禁用的
- `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`，默认是禁用的

Spring Boot也有一些用于简化自定义该行为的特性。

你可以使用当前的environment配置`ObjectMapper`和`XmlMapper`实例。Jackson提供一个扩展套件，可以用来关闭或开启一些特性，你可以用它们配置Jackson以处理不同方面。这些特性在Jackson中是使用6个枚举进行描述的，并被映射到environment的属性上：

|Jackson枚举|Environment属性|
|------|:-------|
|`com.fasterxml.jackson.databind.DeserializationFeature`|`spring.jackson.deserialization.<feature_name>=true|false`|
|`com.fasterxml.jackson.core.JsonGenerator.Feature`|`spring.jackson.generator.<feature_name>=true|false`|
|`com.fasterxml.jackson.databind.MapperFeature`|`spring.jackson.mapper.<feature_name>=true|false`|
|`com.fasterxml.jackson.core.JsonParser.Feature`|`spring.jackson.parser.<feature_name>=true|false`|
|`com.fasterxml.jackson.databind.SerializationFeature`|`spring.jackson.serialization.<feature_name>=true|false`|
|`com.fasterxml.jackson.annotation.JsonInclude.Include`|`spring.jackson.serialization-inclusion=always|non_null|non_absent|non_default|non_empty`|

例如，设置`spring.jackson.serialization.indent_output=true`可以美化打印输出（pretty print）。注意，由于[松散绑定](../IV. Spring Boot features/24.7.2. Relaxed binding.md)的使用，`indent_output`不必匹配对应的枚举常量`INDENT_OUTPUT`。

基于environment的配置会应用到自动配置的`Jackson2ObjectMapperBuilder` bean，然后应用到通过该builder创建的mappers，包括自动配置的`ObjectMapper` bean。

`ApplicationContext`中的`Jackson2ObjectMapperBuilder`可以通过`Jackson2ObjectMapperBuilderCustomizer` bean自定义。这些customizer beans可以排序，Spring Boot自己的customizer序号为0，其他自定义可以应用到Spring Boot自定义之前或之后。

所有类型为`com.fasterxml.jackson.databind.Module`的beans都会自动注册到自动配置的`Jackson2ObjectMapperBuilder`，并应用到它创建的任何`ObjectMapper`实例。这提供了一种全局机制，用于在为应用添加新特性时贡献自定义模块。

如果想完全替换默认的`ObjectMapper`，你既可以定义该类型的`@Bean`并注解`@Primary`，也可以定义`Jackson2ObjectMapperBuilder` `@Bean`，通过builder构建。注意不管哪种方式都会禁用所有的自动配置`ObjectMapper`。

如果你提供`MappingJackson2HttpMessageConverter`类型的`@Bean`，它们将替换MVC配置中的默认值。Spring Boot也提供了一个`HttpMessageConverters`类型的便利bean（如果你使用MVC默认配置，那它就总是可用的），它提供了一些有用的方法来获取默认和用户增强的消息转换器（message converters）。具体详情可参考[Section 71.4, “Customize the @ResponseBody rendering”](./71.4 Customize the @ResponseBody rendering.md)及[WebMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)源码。

### 71.4 自定义@ResponseBody渲染

Spring使用`HttpMessageConverters`渲染`@ResponseBody`（或来自`@RestController`的响应），你可以通过在Spring Boot上下文中添加该类型的beans来贡献其他的转换器。如果你添加的bean类型默认已经包含了（像用于JSON转换的`MappingJackson2HttpMessageConverter`），那它将替换默认的。Spring Boot提供一个方便的`HttpMessageConverters`类型的bean，它有一些有用的方法可以访问默认的和用户增强的message转换器（比如你想要手动将它们注入到一个自定义的`RestTemplate`时就很有用）。

在通常的MVC用例中，任何你提供的`WebMvcConfigurerAdapter` beans通过覆盖`configureMessageConverters`方法也能贡献转换器，但不同于通常的MVC，你可以只提供你需要的转换器（因为Spring Boot使用相同的机制来贡献它默认的转换器）。最终，如果你通过提供自己的` @EnableWebMvc`注解覆盖Spring Boot默认的MVC配置，那你就可以完全控制，并使用来自`WebMvcConfigurationSupport`的`getMessageConverters`手动做任何事。

更多详情可参考[WebMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)源码。

### 71.5 处理Multipart文件上传

Spring Boot采用Servlet 3 `javax.servlet.http.Part` API来支持文件上传。默认情况下，Spring Boot配置Spring MVC在单个请求中只处理每个文件最大1Mb，最多10Mb的文件数据。你可以覆盖那些值，也可以设置临时文件存储的位置（比如，存储到`/tmp`文件夹下）及传递数据刷新到磁盘的阀值（通过使用`MultipartProperties`类暴露的属性）。如果你需要设置文件不受限制，可以设置`spring.http.multipart.max-file-size`属性值为`-1`。

当你想要接收multipart编码文件数据作为Spring MVC控制器（controller）处理方法中被`@RequestParam`注解的`MultipartFile`类型的参数时，multipart支持就非常有用了。

更多详情可参考[MultipartAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/MultipartAutoConfiguration.java)源码。

### 71.6 关闭Spring MVC DispatcherServlet

Spring Boot想要服务来自应用程序root `/`下的所有内容。如果你想将自己的servlet映射到该目录下也是可以的，但当然你可能失去一些Spring Boot MVC特性。为了添加你自己的servlet，并将它映射到root资源，你只需声明一个`Servlet`类型的`@Bean`，并给它特定的bean名称`dispatcherServlet`（如果只想关闭但不替换它，你可以使用该名称创建不同类型的bean）。

### 71.7 关闭默认的MVC配置

完全控制MVC配置的最简单方式是提供你自己的被`@EnableWebMvc`注解的`@Configuration`，这样所有的MVC配置都逃不出你的掌心。

### 71.8 自定义ViewResolvers

`ViewResolver`是Spring MVC的核心组件，它负责转换`@Controller`中的视图名称到实际的`View`实现。注意`ViewResolvers`主要用在UI应用中，而不是REST风格的服务（`View`不是用来渲染`@ResponseBody`的）。Spring有很多你可以选择的`ViewResolver`实现，并且Spring自己对如何选择相应实现也没发表意见。另一方面，Spring Boot会根据classpath上的依赖和应用上下文为你安装一或两个`ViewResolver`实现。`DispatcherServlet`使用所有在应用上下文中找到的解析器（resolvers），并依次尝试每一个直到它获取到结果，所以如果你正在添加自己的解析器，那就要小心顺序和你的解析器添加的位置。

`WebMvcAutoConfiguration`将会为你的上下文添加以下`ViewResolvers`：

- bean id为`defaultViewResolver`的`InternalResourceViewResolver`，它会定位可以使用`DefaultServlet`渲染的物理资源（比如静态资源和JSP页面）。它在视图名上应用了一个前缀和后缀（默认都为空，但你可以通过`spring.view.prefix`和`spring.view.suffix`设置），然后查找在servlet上下文中具有该路径的物理资源，可以通过提供相同类型的bean覆盖它。
- id为`beanNameViewResolver`的`BeanNameViewResolver`，它是视图解析器链的一个非常有用的成员，可以在`View`解析时收集任何具有相同名称的beans，没必要覆盖或替换它。
- id为`viewResolver`的`ContentNegotiatingViewResolver`，它只会在实际`View`类型的beans出现时添加。这是一个'master'解析器，它的职责会代理给其他解析器，它会尝试找到客户端发送的一个匹配'Accept'的HTTP头部。这有一篇关于[ContentNegotiatingViewResolver](https://spring.io/blog/2013/06/03/content-negotiation-using-views)的博客，你也可以也查看下源码。通过定义一个名叫'viewResolver'的bean，你可以关闭自动配置的`ContentNegotiatingViewResolver`。
- 如果使用Thymeleaf，你将有一个id为`thymeleafViewResolver`的`ThymeleafViewResolver`，它会通过加前缀和后缀的视图名来查找资源（外部配置为`spring.thymeleaf.prefix`和`spring.thymeleaf.suffix`，对应的默认为'classpath:/templates/'和'.html'）。你可以通过提供相同名称的bean来覆盖它。
- 如果使用FreeMarker，你将有一个id为`freeMarkerViewResolver`的`FreeMarkerViewResolver`，它会使用加前缀和后缀（外部配置为`spring.freemarker.prefix`和`spring.freemarker.suffix`，对应的默认值为空和'.ftl'）的视图名从加载路径（外部配置为`spring.freemarker.templateLoaderPath`，默认为'classpath:/templates/'）下查找资源。你可以通过提供相同名称的bean来覆盖它。
- 如果使用Groovy模板（实际上只要你把groovy-templates添加到classpath下），你将有一个id为`groovyTemplateViewResolver`的`Groovy TemplateViewResolver`，它会使用加前缀和后缀（外部属性为`spring.groovy.template.prefix`和`spring.groovy.template.suffix`，对应的默认值为'classpath:/templates/'和'.tpl'）的视图名从加载路径下查找资源。你可以通过提供相同名称的bean来覆盖它。
- 如果使用Velocity，你将有一个id为`velocityViewResolver`的`VelocityViewResolver`，它会使用加前缀和后缀（外部属性为`spring.velocity.prefix`和`spring.velocity.suffix`，对应的默认值为空和'.vm'）的视图名从加载路径（外部属性为`spring.velocity.resourceLoaderPath`，默认为'classpath:/templates/'）下查找资源。你可以通过提供相同名称的bean来覆盖它。

更多详情可查看源码：  [WebMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/WebMvcAutoConfiguration.java)，[ThymeleafAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)，[FreeMarkerAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java)，[GroovyTemplateAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java)，[VelocityAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/velocity/VelocityAutoConfiguration.java)。

###     Velocity

默认情况下，Spring Boot会配置一个`VelocityViewResolver`，如果需要的是`VelocityLayoutViewResolver`，你可以自己创建一个名为`velocityViewResolver`的bean。你也可以将`VelocityProperties`实例注入到自定义视图解析器以获取基本的默认设置。

以下示例使用`VelocityLayoutViewResolver`替换自动配置的velocity视图解析器，并自定义`layoutUrl`及应用所有自动配置的属性：
```java
@Bean(name = "velocityViewResolver")
public VelocityLayoutViewResolver velocityViewResolver(VelocityProperties properties) {
    VelocityLayoutViewResolver resolver = new VelocityLayoutViewResolver();
    properties.applyToViewResolver(resolver);
    resolver.setLayoutUrl("layout/default.vm");
    return resolver;
}
```

###    HTTP客户端

###     配置RestTemplate使用代理
正如[Section 33.1, “RestTemplate customization”](../IV. Spring Boot features/33.1 RestTemplate customization.md)描述的那样，你可以使用`RestTemplateCustomizer`和`RestTemplateBuilder`构建一个自定义的`RestTemplate`，这是创建使用代理的`RestTemplate`的推荐方式。

代理配置的确切细节取决于底层使用的客户端请求factory，这里有个示例演示`HttpClient`配置的`HttpComponentsClientRequestFactory`对所有hosts都使用代理，除了`192.168.0.5`。
```java
static class ProxyCustomizer implements RestTemplateCustomizer {

    @Override
    public void customize(RestTemplate restTemplate) {
        HttpHost proxy = new HttpHost("proxy.example.com");
        HttpClient httpClient = HttpClientBuilder.create()
                .setRoutePlanner(new DefaultProxyRoutePlanner(proxy) {

                    @Override
                    public HttpHost determineProxy(HttpHost target,
                            HttpRequest request, HttpContext context)
                                    throws HttpException {
                        if (target.getHostName().equals("192.168.0.5")) {
                            return null;
                        }
                        return super.determineProxy(target, request, context);
                    }

                }).build();
        restTemplate.setRequestFactory(
                new HttpComponentsClientHttpRequestFactory(httpClient));
    }

}
```

### 73. 日志

Spring Boot除了`commons-logging`API外没有其他强制性的日志依赖，你有很多可选的日志实现。想要使用[Logback](http://logback.qos.ch/)，你需要包含它及`jcl-over-slf4j`（它实现了Commons Logging API）。最简单的方式是通过依赖`spring-boot-starter-logging`的starters。对于一个web应用程序，你只需添加`spring-boot-starter-web`依赖，因为它依赖于logging starter。例如，使用Maven：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
Spring Boot有一个`LoggingSystem`抽象，用于尝试通过classpath上下文配置日志系统。如果Logback可用，则首选它。如果你唯一需要做的就是设置不同日志级别，那可以通过在`application.properties`中使用`logging.level`前缀实现，比如：
```java
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```
你也可以使用`logging.file`设置日志文件的位置（除控制台之外，默认会输出到控制台）。

想要对日志系统进行更细粒度的配置，你需要使用`LoggingSystem`支持的原生配置格式。默认情况下，Spring Boot从系统的默认位置加载原生配置（比如对于Logback为`classpath:logback.xml`），但你可以使用`logging.config`属性设置配置文件的位置。

### 73.1 配置Logback

如果你将`logback.xml`放到classpath根目录下，那它将会被从这加载（或`logback-spring.xml`充分利用Boot提供的模板特性）。Spring Boot提供一个默认的基本配置，如果你只是设置日志级别，那你可以包含它，比如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <logger name="org.springframework.web" level="DEBUG"/>
</configuration>
```
如果查看spring-boot jar中的`base.xml`，你将会看到`LoggingSystem`为你创建的很多有用的系统属性，比如：
- `${PID}`，当前进程id。
- `${LOG_FILE}`，如果在Boot外部配置中设置了`logging.file`。
- `${LOG_PATH}`，如果设置了`logging.path`（表示日志文件产生的目录）。
- `${LOG_EXCEPTION_CONVERSION_WORD}`，如果在Boot外部配置中设置了`logging.exception-conversion-word`。

Spring Boot也提供使用自定义的Logback转换器在控制台上输出一些漂亮的彩色ANSI日志信息（不是日志文件），具体参考默认的`base.xml`配置。

如果Groovy在classpath下，你也可以使用`logback.groovy`配置Logback。

###       配置logback只输出到文件

如果想禁用控制台日志记录，只将输出写入文件中，你需要一个只导入`file-appender.xml`而不是`console-appender.xml`的自定义`logback-spring.xml`：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}/}spring.log}"/>
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <root level="INFO">
        <appender-ref ref="FILE" />
    </root>
</configuration>
```
你还需要将`logging.file`添加到`application.properties`：
```properties
logging.file=myapplication.log
```

### 73.2 配置Log4j

如果[Log4j 2](http://logging.apache.org/log4j/2.x)出现在classpath下，Spring Boot会将其作为日志配置。如果你正在使用starters进行依赖装配，这意味着你需要排除Logback，然后包含log4j 2。如果不使用starters，除了添加Log4j 2，你还需要提供`jcl-over-slf4j`依赖（至少）。

最简单的方式可能就是通过starters，尽管它需要排除一些依赖，比如，在Maven中：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

**注** Log4j starters会收集好依赖以满足普通日志记录的需求（比如，Tomcat中使用`java.util.logging`，但使用Log4j 2作为输出），具体查看Actuator Log4j 2的示例，了解如何将它用于实战。

### 73.2.1 使用YAML或JSON配置Log4j2

除了它的默认XML配置格式，Log4j 2也支持YAML和JSON配置文件。想使用其他配置文件格式配置Log4j 2，你需要添加合适的依赖到classpath，并以匹配所选格式的方式命名配置文件：

|格式|依赖|文件名|
|:----|:----|:---|
|YAML|`com.fasterxml.jackson.core:jackson-databind` `com.fasterxml.jackson.dataformat:jackson-dataformat-yaml`|`log4j2.yaml` `log4j2.yml`|
|JSON|`com.fasterxml.jackson.core:jackson-databind`|`log4j2.json` `log4j2.jsn`|

### 74. 数据访问

### 74.1 配置数据源

自定义`DataSource`类型的`@Bean`可以覆盖默认设置，正如[Section 24.7.1, “Third-party configuration”](../IV. Spring Boot features/24.7.1. Third-party configuration.md)解释的那样，你可以很轻松的将它跟一系列`Environment`属性绑定：
```java
@Bean
@ConfigurationProperties(prefix="datasource.fancy")
public DataSource dataSource() {
    return new FancyDataSource();
}
```
```properties
datasource.fancy.jdbcUrl=jdbc:h2:mem:mydb
datasource.fancy.username=sa
datasource.fancy.poolSize=30
```
Spring Boot也提供了一个工具类`DataSourceBuilder`用来创建标准的数据源。如果需要重用`DataSourceProperties`的配置，你可以从它初始化一个`DataSourceBuilder`：
```java
@Bean
@ConfigurationProperties(prefix="datasource.mine")
public DataSource dataSource(DataSourceProperties properties) {
    return properties.initializeDataSourceBuilder()
            // additional customizations
            .build();
}
```
在此场景中，你保留了通过Spring Boot暴露的标准属性，通过添加`@ConfigurationProperties`，你可以暴露在相应的命命名空间暴露其他特定实现的配置，
具体详情可参考'Spring Boot特性'章节中的[Section 29.1, “Configure a DataSource”](../IV. Spring Boot features/29.1. Configure a DataSource.md)和[DataSourceAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.java)类源码。

### 74.10 将Spring Data仓库暴露为REST端点

Spring Data REST能够将`Repository`的实现暴露为REST端点，只要该应用启用Spring MVC。Spring Boot暴露一系列来自`spring.data.rest`命名空间的有用属性来定制化[RepositoryRestConfiguration](http://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/core/config/RepositoryRestConfiguration.html)，你可以使用[`RepositoryRestConfigurer`](http://docs.spring.io/spring-data/rest/docs/current/api/org/springframework/data/rest/webmvc/config/RepositoryRestConfigurer.html)提供其他定制。

###      配置JPA使用的组件

如果想配置一个JPA使用的组件，你需要确保该组件在JPA之前初始化。组件如果是Spring Boot自动配置的，Spring Boot会为你处理。例如，Flyway是自动配置的，Hibernate依赖于Flyway，这样Hibernate有机会在使用数据库前对其进行初始化。

如果自己配置组件，你可以使用`EntityManagerFactoryDependsOnPostProcessor`子类设置必要的依赖，例如，如果你正使用Hibernate搜索，并将Elasticsearch作为它的索引管理器，这样任何`EntityManagerFactory` beans必须设置为依赖`elasticsearchClient` bean：
```java
/**
 * {@link EntityManagerFactoryDependsOnPostProcessor} that ensures that
 * {@link EntityManagerFactory} beans depend on the {@code elasticsearchClient} bean.
 */
@Configuration
static class ElasticsearchJpaDependencyConfiguration
        extends EntityManagerFactoryDependsOnPostProcessor {

    ElasticsearchJpaDependencyConfiguration() {
        super("elasticsearchClient");
    }

}
```

### 74.2 配置两个数据源

创建多个数据源和创建一个工作都是一样的，如果使用JDBC或JPA的默认自动配置，你需要将其中一个设置为`@Primary`（然后它就能被任何`@Autowired`注入获取）。
```java
@Bean
@Primary
@ConfigurationProperties(prefix="datasource.primary")
public DataSource primaryDataSource() {
    return DataSourceBuilder.create().build();
}

@Bean
@ConfigurationProperties(prefix="datasource.secondary")
public DataSource secondaryDataSource() {
    return DataSourceBuilder.create().build();
}
```

### 74.3 使用Spring Data仓库

Spring Data可以为你的`@Repository`接口创建各种风格的实现。Spring Boot会为你处理所有事情，只要那些`@Repositories`接口跟你的`@EnableAutoConfiguration`类处于相同的包（或子包）。

对于很多应用来说，你需要做的就是将正确的Spring Data依赖添加到classpath下（JPA对应`spring-boot-starter-data-jpa`，Mongodb对应`spring-boot-starter-data-mongodb`），创建一些repository接口来处理`@Entity`对象，相应示例可参考[JPA sample](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-samples/spring-boot-sample-data-jpa)或[Mongodb sample](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-samples/spring-boot-sample-data-mongodb)。

Spring Boot会基于它找到的`@EnableAutoConfiguration`来尝试猜测你的`@Repository`定义的位置。想要获取更多控制，可以使用`@EnableJpaRepositories`注解（来自Spring Data JPA）。

### 74.4 从Spring配置分离`@Entity`定义

Spring Boot会基于它找到的`@EnableAutoConfiguration`来尝试猜测`@Entity`定义的位置，想要获取更多控制可以使用`@EntityScan`注解，比如：
```java
@Configuration
@EnableAutoConfiguration
@EntityScan(basePackageClasses=City.class)
public class Application {

    //...

}
```

### 74.5 配置JPA属性

Spring Data JPA已经提供了一些独立的配置选项（比如，针对SQL日志），并且Spring Boot会暴露它们，针对hibernate的外部配置属性也更多些，最常见的选项如下：
```java
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.hibernate.naming.physical-strategy=com.example.MyPhysicalNamingStrategy
spring.jpa.database=H2
spring.jpa.show-sql=true
```
`ddl-auto`配置是个特殊情况，它的默认设置取决于是否使用内嵌数据库（是则默认值为`create-drop`，否则为`none`）。当本地`EntityManagerFactory`被创建时，所有`spring.jpa.properties.*`属性都被作为正常的JPA属性（去掉前缀）传递进去了。

Spring Boot提供一致的命名策略，不管你使用什么Hibernate版本。如果使用Hibernate 4，你可以使用`spring.jpa.hibernate.naming.strategy`进行自定义；Hibernate 5定义一个`Physical`和`Implicit`命名策略：Spring Boot默认配置`SpringPhysicalNamingStrategy`，该实现提供跟Hibernate 4相同的表结构。如果你情愿使用Hibernate 5默认的，可以设置以下属性：
```properties
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```
具体详情可参考[HibernateJpaAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.java)和[JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)。

### 74.6 使用自定义EntityManagerFactory

为了完全控制`EntityManagerFactory`的配置，你需要添加一个名为`entityManagerFactory`的`@Bean`，Spring Boot自动配置会根据是否存在该类型的bean来关闭它的实体管理器（entity manager）。

### 74.7 使用两个EntityManagers

即使默认的`EntityManagerFactory`工作的很好，你也需要定义一个新的`EntityManagerFactory`，因为一旦出现第二个该类型的bean，默认的将会被关闭。为了轻松的实现该操作，你可以使用Spring Boot提供的`EntityManagerBuilder`，或者如果你喜欢的话可以直接使用来自Spring ORM的`LocalContainerEntityManagerFactoryBean`。

示例：
```java
// add two data sources configured as above

@Bean
public LocalContainerEntityManagerFactoryBean customerEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(customerDataSource())
            .packages(Customer.class)
            .persistenceUnit("customers")
            .build();
}

@Bean
public LocalContainerEntityManagerFactoryBean orderEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(orderDataSource())
            .packages(Order.class)
            .persistenceUnit("orders")
            .build();
}
```
上面的配置靠自己基本可以运行，想要完成作品你还需要为两个`EntityManagers`配置`TransactionManagers`。其中的一个会被Spring Boot默认的`JpaTransactionManager`获取，如果你将它标记为`@Primary`。另一个需要显式注入到一个新实例。或你可以使用一个JTA事物管理器生成它两个。

如果使用Spring Data，你需要相应地需要配置`@EnableJpaRepositories`：
```java
@Configuration
@EnableJpaRepositories(basePackageClasses = Customer.class,
        entityManagerFactoryRef = "customerEntityManagerFactory")
public class CustomerConfiguration {
    ...
}

@Configuration
@EnableJpaRepositories(basePackageClasses = Order.class,
        entityManagerFactoryRef = "orderEntityManagerFactory")
public class OrderConfiguration {
    ...
}
```

### 74.8 使用普通的persistence.xml

Spring不要求使用XML配置JPA提供者（provider），并且Spring Boot假定你想要充分利用该特性。如果你倾向于使用`persistence.xml`，那你需要定义你自己的id为`entityManagerFactory`的`LocalEntityManagerFactoryBean`类型的`@Bean`，并在那设置持久化单元的名称，默认设置可查看[JpaBaseConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)。

### 74.9 使用Spring Data JPA和Mongo仓库

Spring Data JPA和Spring Data Mongo都能自动为你创建`Repository`实现。如果它们同时出现在classpath下，你可能需要添加额外的配置来告诉Spring Boot你想要哪个（或两个）为你创建仓库。最明确地方式是使用标准的Spring Data `@Enable*Repositories`，然后告诉它你的`Repository`接口的位置（此处`*`即可以是Jpa，也可以是Mongo，或者两者都是）。

这里也有`spring.data.*.repositories.enabled`标志，可用来在外部配置中开启或关闭仓库的自动配置，这在你想关闭Mongo仓库但仍使用自动配置的`MongoTemplate`时非常有用。

相同的障碍和特性也存在于其他自动配置的Spring Data仓库类型（Elasticsearch, Solr），只需要改变对应注解的名称和标志。

### 75. 数据库初始化

一个数据库可以使用不同的方式进行初始化，这取决于你的技术栈。或者你可以手动完成该任务，只要数据库是单独的过程。

### 75.1 使用JPA初始化数据库

JPA有个生成DDL的特性，并且可以设置为在数据库启动时运行，这可以通过两个外部属性进行控制：

- `spring.jpa.generate-ddl`（`boolean`）控制该特性的关闭和开启，跟实现者没关系。
- `spring.jpa.hibernate.ddl-auto`（`enum`）是一个Hibernate特性，用于更细力度的控制该行为，更多详情参考以下内容。

### 75.2 使用Hibernate初始化数据库

你可以显式设置`spring.jpa.hibernate.ddl-auto`，标准的Hibernate属性值有`none`，`validate`，`update`，`create`，`create-drop`。Spring Boot根据你的数据库是否为内嵌数据库来选择相应的默认值，如果是内嵌型的则默认值为`create-drop`，否则为`none`。通过查看`Connection`类型可以检查是否为内嵌型数据库，hsqldb，h2和derby是内嵌的，其他都不是。当从内存数据库迁移到一个真正的数据库时，你需要当心，在新的平台中不能对数据库表和数据是否存在进行臆断，你也需要显式设置`ddl-auto`，或使用其他机制初始化数据库。

此外，启动时处于classpath根目录下的`import.sql`文件会被执行。这在demos或测试时很有用，但在生产环境中你可能不期望这样。这是Hibernate的特性，和Spring没有一点关系。

### 75.3 使用Spring JDBC初始化数据库

Spring JDBC有一个初始化`DataSource`特性，Spring Boot默认启用该特性，并从标准的位置`schema.sql`和`data.sql`（位于classpath根目录）加载SQL。此外，Spring Boot将加载`schema-${platform}.sql`和`data-${platform}.sql`文件（如果存在），在这里`platform`是`spring.datasource.platform`的值，比如，你可以将它设置为数据库的供应商名称（`hsqldb`, `h2`, `oracle`, `mysql`, `postgresql`等）。Spring Boot默认启用Spring JDBC初始化快速失败特性，所以如果脚本导致异常产生，那应用程序将启动失败。脚本的位置可以通过设置`spring.datasource.schema`和`spring.datasource.data`来改变，如果设置`spring.datasource.initialize=false`则哪个位置都不会被处理。

你可以设置`spring.datasource.continue-on-error=true`禁用快速失败特性。一旦应用程序成熟并被部署了很多次，那该设置就很有用，因为脚本可以充当"可怜人的迁移"-例如，插入失败时意味着数据已经存在，也就没必要阻止应用继续运行。

如果你想要在一个JPA应用中使用`schema.sql`，那如果Hibernate试图创建相同的表，`ddl-auto=create-drop`将导致错误产生。为了避免那些错误，可以将`ddl-auto`设置为“”（推荐）或`none`。不管是否使用`ddl-auto=create-drop`，你总可以使用`data.sql`初始化新数据。

### 75.4 初始化Spring Batch数据库

如果你正在使用Spring Batch，那么它会为大多数的流行数据库平台预装SQL初始化脚本。Spring Boot会检测你的数据库类型，并默认执行那些脚本，在这种情况下将关闭快速失败特性（错误被记录但不会阻止应用启动）。这是因为那些脚本是可信任的，通常不会包含bugs，所以错误会被忽略掉，并且对错误的忽略可以让脚本具有幂等性。你可以使用`spring.batch.initializer.enabled=false`显式关闭初始化功能。

### 75.5 使用高级数据迁移工具

Spring Boot支持两种高级数据迁移工具[Flyway](http://flywaydb.org/)(基于SQL)和[Liquibase](http://www.liquibase.org/)(XML)。

### 75.5.1 启动时执行Flyway数据库迁移

想要在启动时自动运行Flyway数据库迁移，需要将`org.flywaydb:flyway-core`添加到你的classpath下。

迁移是一些`V<VERSION>__<NAME>.sql`格式的脚本（`<VERSION>`是一个下划线分割的版本号，比如'1'或'2_1'）。默认情况下，它们存放在`classpath:db/migration`文件夹中，但你可以使用`flyway.locations`（一个列表）改变它。详情可参考flyway-core中的`Flyway`类，查看一些可用的配置，比如schemas。Spring Boot在[FlywayProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayProperties.java)中提供了一个小的属性集，可用于禁止迁移，或关闭位置检测。Spring Boot将调用`Flyway.migrate()`执行数据库迁移，如果想要更多控制可提供一个实现[FlywayMigrationStrategy](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayMigrationStrategy.java)的`@Bean`。

默认情况下，Flyway将自动注入（`@Primary`）`DataSource`到你的上下文，并用它进行数据迁移。如果想使用不同的`DataSource`，你可以创建一个，并将它标记为`@FlywayDataSource`的`@Bean`-如果你这样做了，且想要两个数据源，记得创建另一个并将它标记为`@Primary`，或者你可以通过在外部配置文件中设置`flyway.[url,user,password]`来使用Flyway的原生`DataSource`。

这是一个[Flyway示例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-flyway)，你可以作为参考。

### 75.5.2 启动时执行Liquibase数据库迁移

想要在启动时自动运行Liquibase数据库迁移，你需要将`org.liquibase:liquibase-core`添加到classpath下。

你可以使用`liquibase.change-log`设置master变化日志位置，默认从`db/changelog/db.changelog-master.yaml`读取。除了YAML，Liquibase还支持JSON, XML和SQL改变日志格式。查看[LiquibaseProperties](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/liquibase/LiquibaseProperties.java)获取可用配置，比如上下文，默认schema等。


这里有个[Liquibase示例](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-liquibase)可作为参考。

### 76. 批处理应用

### 76.1 在启动时执行Spring Batch作业

你可以在上下文的某个地方添加`@EnableBatchProcessing`来启用Spring Batch的自动配置功能。

默认情况下，在启动时它会执行应用的所有作业（Jobs），具体查看[JobLauncherCommandLineRunner](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/JobLauncherCommandLineRunner.java)。你可以通过指定`spring.batch.job.names`（多个作业名以逗号分割）来缩小到一个特定的作业或多个作业。

如果应用上下文包含一个`JobRegistry`，那么处于`spring.batch.job.names`中的作业将会从registry中查找，而不是从上下文中自动装配。这是复杂系统中常见的一个模式，在这些系统中多个作业被定义在子上下文和注册中心。

详情可参考[BatchAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java)和[@EnableBatchProcessing](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.java)。

### 77. 执行器（Actuator）


### 77.1 改变HTTP端口或执行器端点的地址

在一个单独的应用中，执行器的HTTP端口默认和主HTTP端口相同。想要让应用监听不同的端口，你可以设置外部属性`management.port`。为了监听一个完全不同的网络地址（比如，你有一个用于管理的内部网络和一个用于用户应用程序的外部网络），你可以将`management.address`设置为一个可用的IP地址，然后将服务器绑定到该地址。

更多详情可查看[ManagementServerProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/autoconfigure/ManagementServerProperties.java)源码和'Production-ready特性'章节中的[Section 47.3, “Customizing the management server port”](../V. Spring Boot Actuator/47.3 Customizing the management server port.md)。

### 77.2 自定义WhiteLabel错误页面

Spring Boot安装了一个'whitelabel'错误页面，如果你遇到一个服务器错误（机器客户端消费的是JSON，其他媒体类型则会看到一个具有正确错误码的合乎情理的响应），那就能在客户端浏览器中看到该页面。你可以设置`error.whitelabel.enabled=false`来关闭该功能，但通常你想要添加自己的错误页面来取代whitelabel。确切地说，如何实现取决于你使用的模板技术。例如，你正在使用Thymeleaf，你将添加一个`error.html`模板。如果你正在使用FreeMarker，那你将添加一个`error.ftl`模板。通常，你需要的只是一个名称为`error`的`View`，或一个处理`/error`路径的`@Controller`。除非你替换了一些默认配置，否则你将在你的`ApplicationContext`中找到一个`BeanNameViewResolver`，所以一个id为`error`的`@Bean`可能是完成该操作的一个简单方式，详情可参考[ErrorMvcAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ErrorMvcAutoConfiguration.java)。

查看[Error Handling](../IV. Spring Boot features/27.1.8 Error Handling.md)章节，了解如何将处理器（handlers）注册到servlet容器中。

###     Actuator和Jersey

执行器HTTP端点只有在基于Spring MVC的应用才可用，如果想使用Jersey和执行器，你需要启用Spring MVC（添加`spring-boot-starter-web`依赖）。默认情况下，Jersey和 Spring MVC分发器servlet被映射到相同路径（`/`）。你需要改变它们中的某个路径（Spring MVC可以配置`server.servlet-path`，Jersey可以配置`spring.jersey.application-path`）。例如，如果你在`application.properties`中添加`server.servlet-path=/system`，你将在`/system`访问执行器HTTP端点。

### 78. 安全

### 78.1 关闭Spring Boot安全配置

不管你在应用的什么地方定义了一个使用`@EnableWebSecurity`注解的`@Configuration`，它都会关闭Spring Boot中的默认webapp安全设置。想要调整默认值，你可以尝试设置`security.*`属性（具体查看[SecurityProperties](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/security/SecurityProperties.java)和[常见应用属性](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#common-application-properties-security)的SECURITY章节）。

### 78.2 改变AuthenticationManager并添加用户账号

如果你提供了一个`AuthenticationManager`类型的`@Bean`，那么默认的就不会被创建了，所以你可以获得Spring Security可用的全部特性（比如，[不同的认证选项](http://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication)）。

Spring Security也提供了一个方便的`AuthenticationManagerBuilder`，用于构建具有常见选项的`AuthenticationManager`。在一个webapp中，推荐将它注入到`WebSecurityConfigurerAdapter`的一个void方法中，比如：
```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
            auth.inMemoryAuthentication()
                .withUser("barry").password("password").roles("USER"); // ... etc.
    }

    // ... other stuff for application security
}
```
如果把它放到一个内部类或一个单独的类中，你将得到最好的结果（也就是不跟很多其他`@Beans`混合在一起将允许你改变实例化的顺序）。[secure web sample](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-web-secure)是一个有用的参考模板。

如果你遇到了实例化问题（比如，使用JDBC或JPA进行用户详细信息的存储），那将`AuthenticationManagerBuilder`回调提取到一个`GlobalAuthenticationConfigurerAdapter`（放到`init()`方法内以防其他地方也需要authentication manager）可能是个不错的选择，比如：
```java
@Configuration
public class AuthenticationManagerConfiguration extends

    GlobalAuthenticationConfigurerAdapter {
    @Override
    public void init(AuthenticationManagerBuilder auth) {
        auth.inMemoryAuthentication() // ... etc.
    }

}
```

### 78.3 当前端使用代理服务器时启用HTTPS

对于任何应用来说，确保所有的主端点（URL）都只在HTTPS下可用是个重要的苦差事。如果你使用Tomcat作为servlet容器，那Spring Boot如果发现一些环境设置的话，它将自动添加Tomcat自己的`RemoteIpValve`，你也可以依赖于`HttpServletRequest`来报告是否请求是安全的（即使代理服务器的downstream处理真实的SSL终端）。这个标准行为取决于某些请求头是否出现（`x-forwarded-for`和`x-forwarded-proto`），这些请求头的名称都是约定好的，所以对于大多数前端和代理都是有效的。

你可以向`application.properties`添加以下设置开启该功能，比如：
```yml
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto
```
（这些属性出现一个就会开启该功能，或者你可以通过添加一个`TomcatEmbeddedServletContainerFactory` bean自己添加`RemoteIpValve`）。

Spring Security也可以配置成针对所有或某些请求需要一个安全渠道（channel）。想要在一个Spring Boot应用中开启它，你只需将`application.properties`中的`security.require_ssl`设置为`true`即可。

### 79. 热交换

### 79.1 重新加载静态内容

Spring Boot有很多用于热加载的选项，不过推荐使用[spring-boot-devtools](../III. Using Spring Boot/20. Developer tools.md)，因为它提供了其他开发时特性，比如快速应用重启和LiveReload，还有开发时敏感的配置加载（比如，模板缓存）。


此外，使用IDE开发也是一个不错的方式，特别是需要调试的时候（所有的现代IDEs都允许重新加载静态资源，通常也支持对变更的Java类进行热交换）。

最后，[Maven和Gradle插件](../VIII. Build tool plugins/README.md)也支持命令行下的静态文件热加载。如果你使用其他高级工具编写css/js，并使用外部的css/js编译器，那你就可以充分利用该功能。

###      在不重启容器的情况下重新加载模板

Spring Boot支持的大多数模板技术包含一个禁用缓存的配置选项，如果你正在使用`spring-boot-devtools`模块，Spring Boot在开发期间会自动为你[配置那些属性](../III. Using Spring Boot/20.1 Property defaults.md)。

### 79.2.1 Thymeleaf模板

如果你正在使用Thymeleaf，那就将`spring.thymeleaf.cache`设置为`false`，查看[ThymeleafAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)可以获取其他Thymeleaf自定义选项。

### 79.2.2 FreeMarker模板

如果你正在使用FreeMarker，那就将`spring.freemarker.cache`设置为`false`，查看[FreeMarkerAutoConfiguration ](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java)可以获取其他FreeMarker自定义选项。

### 79.2.3 Groovy模板

如果你正在使用Groovy模板，那就将`spring.groovy.template.cache`设置为`false`，查看[GroovyTemplateAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java)可以获取其他Groovy自定义选项。

### 79.2.4 Velocity模板

如果你正在使用Velocity，那就将`spring.velocity.cache`设置为`false`，查看[VelocityAutoConfiguration](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/velocity/VelocityAutoConfiguration.java)可以获取其他Velocity自定义选项。

###     应用快速重启

`spring-boot-devtools`模块包括应用自动重启支持，虽然没有其他技术快，比如[JRebel](http://zeroturnaround.com/software/jrebel/)或[Spring Loaded](https://github.com/spring-projects/spring-loaded)，但比"冷启动"快。在研究其他复杂重启选项时，你最好自己先试下，更多详情可参考[Chapter 20, Developer tools](../III. Using Spring Boot/20. Developer tools.md)章节。

### 79.4 在不重启容器的情况下重新加载Java类

现代IDEs（Eclipse, IDEA等）都支持字节码的热交换，所以如果你做了一个没有影响类或方法签名的改变，它会利索地重新加载并没有任何影响。

[Spring Loaded](https://github.com/spring-projects/spring-loaded)在这方面走的更远，它能够重新加载方法签名改变的类定义，如果对它进行一些自定义配置可以强制`ApplicationContext`刷新自己（但没有通用的机制来确保这对一个运行中的应用总是安全的，所以它可能只是一个开发时的技巧）。

### 79.4.1 使用Maven配置Spring Loaded

为了在Maven命令行下使用Spring Loaded，你只需将它作为依赖添加到Spring Boot插件声明中即可，比如：
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>springloaded</artifactId>
            <version>1.2.0.RELEASE</version>
        </dependency>
    </dependencies>
</plugin>
```
正常情况下，这在Eclipse和IntelliJ IDEA中工作的相当漂亮，只要它们有相应的，和Maven默认一致的构建配置（Eclipse m2e对此支持的更好，开箱即用）。

### 79.4.2 使用Gradle和IntelliJ IDEA配置Spring Loaded

如果想将Spring Loaded和Gradle，IntelliJ IDEA结合起来，那你需要付出代价。默认情况下，IntelliJ IDEA将类编译到一个跟Gradle不同的位置，这会导致Spring Loaded监控失败。

为了正确配置IntelliJ IDEA，你可以使用`idea` Gradle插件：
```gradle
buildscript {
    repositories { jcenter() }
    dependencies {
        classpath "org.springframework.boot:spring-boot-gradle-plugin:1.4.1.RELEASE"
        classpath 'org.springframework:springloaded:1.2.0.RELEASE'
    }
}

apply plugin: 'idea'

idea {
    module {
        inheritOutputDirs = false
        outputDir = file("$buildDir/classes/main/")
    }
}

// ...
```
**注** IntelliJ IDEA必须配置跟命令行Gradle任务相同的Java版本，并且`springloaded`必须作为一个`buildscript`依赖被包含进去。

此外，你也可以启用Intellij IDEA内部的`Make Project Automatically`，这样不管什么时候只要文件被保存都会自动编译。

### 80. 构建

###     生成构建信息

Maven和Gradle都支持产生包含项目版本，坐标，名称的构建信息，该插件可以通过配置添加其他属性。当这些文件出现时，Spring Boot自动配置一个`BuildProperties` bean。

为了让Maven生成构建信息，你需要为`build-info` goal添加一个execution：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>1.4.1.RELEASE</version>
            <executions>
                <execution>
                    <goals>
                        <goal>build-info</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
**注** 更多详情查看[Spring Boot Maven插件文档](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/)。

使用Gradle实现同样效果：
```gradle
springBoot  {
    buildInfo()
}
```
可以使用DSL添加其他属性：
```gradle
springBoot  {
    buildInfo {
        additionalProperties = [
            'foo': 'bar'
        ]
    }
}
```

### 80.10 使用Ant构建可执行存档（不使用spring-boot-antlib）

想要使用Ant进行构建，你需要抓取依赖，编译，然后像通常那样创建一个jar或war存档。为了让它可以执行，你可以使用`spring-boot-antlib`，也可以使用以下指令：

1. 如果构建jar，你需要将应用的类和资源打包进内嵌的`BOOT-INF/classes`目录。如果构建war，你需要将应用的类打包进内嵌的`WEB-INF/classes`目录。
2. 对于jar，添加运行时依赖到内嵌的`BOOT-INF/lib`目录。对于war，则添加到`WEB-INF/lib`目录。注意不能压缩存档中的实体。
3. 对于jar，添加`provided`依赖到内嵌的`BOOT-INF/lib`目录。对于war，则添加到`WEB-INF/lib-provided`目录。注意不能压缩存档中的实体。
4. 在存档的根目录添加`spring-boot-loader`类（这样`Main-Class`就可用了）。
5. 使用恰当的启动器，比如对于jar使用`JarLauncher`作为manifest的`Main-Class`属性，指定manifest的其他属性，特别是`Start-Class`。

示例：
```xml
<target name="build" depends="compile">
    <jar destfile="target/${ant.project.name}-${spring-boot.version}.jar" compress="false">
        <mappedresources>
            <fileset dir="target/classes" />
            <globmapper from="*" to="BOOT-INF/classes/*"/>
        </mappedresources>
        <mappedresources>
            <fileset dir="src/main/resources" erroronmissingdir="false"/>
            <globmapper from="*" to="BOOT-INF/classes/*"/>
        </mappedresources>
        <mappedresources>
            <fileset dir="${lib.dir}/runtime" />
            <globmapper from="*" to="BOOT-INF/lib/*"/>
        </mappedresources>
        <zipfileset src="${lib.dir}/loader/spring-boot-loader-jar-${spring-boot.version}.jar" />
        <manifest>
            <attribute name="Main-Class" value="org.springframework.boot.loader.JarLauncher" />
            <attribute name="Start-Class" value="${start-class}" />
        </manifest>
    </jar>
</target>
```
该[Ant示例](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-samples/spring-boot-sample-ant)中有一个`build.xml`文件及`manual`任务，可以使用以下命令来运行：
```shell
$ ant -lib <folder containing ivy-2.2.jar> clean manual
```
在上述操作之后，你可以使用以下命令运行该应用：
```shell
$ java -jar target/*.jar
```

### 80.11 如何使用Java6

如果想在Java6环境中使用Spring Boot，你需要改变一些配置，具体的改变取决于你应用的功能。

### 80.11.1 内嵌Servlet容器兼容性

如果你在使用Boot的内嵌Servlet容器，你需要使用一个兼容Java6的容器。Tomcat 7和Jetty 8都是Java 6兼容的。具体参考[Section 70.16 使用Tomcat 7.x或8.0](../IX. ‘How-to’ guides/70.16 Use Tomcat 7.x or 8.0.md)和[Section 70.18 使用Jetty 8](../IX. ‘How-to’ guides/70.18 Use Jetty 8.md)。

###        Jackson

Jackson 2.7及以后版本需要Java 7，如果想要在Java 6环境使用Jackson，你需要降级使用Jackson 2.6。

###        JTA API兼容性

虽然Java Transaction API自身不要求Java 7，但官方API jar包含的已构建类需要Java 7。如果正在使用JTA，你需要使用能够在Java 6环境工作的jar替换官方的JTA 1.2 API jar。想要实现这样的效果，你需要排除任何`javax.transaction:javax.transaction-api`依赖，并使用`org.jboss.spec.javax.transaction:jboss-transaction-api_1.2_spec:1.0.0.Final`替换它。

###     生成Git信息

Maven和Gradle都支持生成一个`git.properties`文件，该文件包含项目构建时`git`源码的仓库状态。对于Maven用户来说，`spring-boot-starter-parent` POM包含一个预配置的插件去产生一个`git.properties`文件，只需简单的将以下声明添加到POM中：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
Gradle用户可以使用[gradle-git-properties](https://plugins.gradle.org/plugin/com.gorylenko.gradle-git-properties)插件实现相同效果：
```gralde
plugins {
    id "com.gorylenko.gradle-git-properties" version "1.4.6"
}
```

### 80.3 自定义依赖版本

如果你使用Maven进行一个直接或间接继承`spring-boot-dependencies`（比如`spring-boot-starter-parent`）的构建，并想覆盖一个特定的第三方依赖，那你可以添加合适的`<properties>`元素。浏览[spring-boot-dependencies](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-dependencies/pom.xml) POM可以获取一个全面的属性列表。例如，想要选择一个不同的`slf4j`版本，你可以添加以下内容：
```xml
<properties>
    <slf4j.version>1.7.5<slf4j.version>
</properties>
```
**注** 这只在你的Maven项目继承（直接或间接）自`spring-boot-dependencies`才有用。如果你使用`<scope>import</scope>`，将`spring-boot-dependencies`添加到自己的`dependencyManagement`片段，那你必须自己重新定义artifact而不是覆盖属性。

**注** 每个Spring Boot发布都是基于一些特定的第三方依赖集进行设计和测试的，覆盖版本可能导致兼容性问题。

Gradle中为了覆盖依赖版本，你需要指定如下所示的version：
```gradle
ext['slf4j.version'] = '1.7.5'
```
更多详情查看[Gradle Dependency Management插件文档](https://github.com/spring-gradle-plugins/dependency-management-plugin)。

### 80.4 使用Maven创建可执行JAR

`spring-boot-maven-plugin`能够用来创建可执行的'胖'JAR。如果正在使用`spring-boot-starter-parent` POM，你可以简单地声明该插件，然后你的jar将被重新打包：
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
如果没有使用parent POM，你仍旧可以使用该插件。不过，你需要另外添加一个`<executions>`片段：
```xml
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
```
查看[插件文档](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/usage.html)获取详细的用例。

###     将Spring Boot应用作为依赖

跟war包一样，Spring Boot应用不是用来作为依赖的。如果你的应用包含需要跟其他项目共享的类，最好的方式是将代码放到单独的模块，然后其他项目及你的应用都可以依赖该模块。

如果不能按照上述推荐的方式重新组织代码，你需要配置Spring Boot的Maven和Gradle插件去产生一个单独的artifact，以适合于作为依赖。可执行存档不能用于依赖，因为[可执行jar格式](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#executable-jar-jar-file-structure)将应用class打包到`BOOT-INF/classes`，也就意味着可执行jar用于依赖时会找不到。

为了产生两个artifacts（一个用于依赖，一个用于可执行jar），你需要指定classifier。classifier用于可执行存档的name，默认存档用于依赖。

可以使用以下配置Maven中classifier的`exec`：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </plugin>
    </plugins>
</build>
```
使用Gradle可以添加以下配置：
```gradle
bootRepackage  {
    classifier = 'exec'
}
```

### 80.6 在可执行jar运行时提取特定的版本

在一个可执行jar中，为了运行，多数内嵌的库不需要拆包（unpacked），然而有一些库可能会遇到问题。例如，JRuby包含它自己的内嵌jar，它假定`jruby-complete.jar`本身总是能够直接作为文件访问的。

为了处理任何有问题的库，你可以标记那些特定的内嵌jars，让它们在可执行jar第一次运行时自动解压到一个临时文件夹中。例如，为了将JRuby标记为使用Maven插件拆包，你需要添加如下的配置：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <requiresUnpack>
                    <dependency>
                        <groupId>org.jruby</groupId>
                        <artifactId>jruby-complete</artifactId>
                    </dependency>
                </requiresUnpack>
            </configuration>
        </plugin>
    </plugins>
</build>
```
使用Gradle完全上述操作：
```gradle
springBoot  {
    requiresUnpack = ['org.jruby:jruby-complete']
}
```

### 80.7 使用排除创建不可执行的JAR

如果你构建的产物既有可执行的jar和非可执行的jar，那你常常需要为可执行的版本添加额外的配置文件，而这些文件在一个library jar中是不需要的。比如，`application.yml`配置文件可能需要从非可执行的JAR中排除。

下面是如何在Maven中实现：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-jar-plugin</artifactId>
            <executions>
                <execution>
                    <id>exec</id>
                    <phase>package</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                    <configuration>
                        <classifier>exec</classifier>
                    </configuration>
                </execution>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                    <configuration>
                        <!-- Need this to ensure application.yml is excluded -->
                        <forceCreation>true</forceCreation>
                        <excludes>
                            <exclude>application.yml</exclude>
                        </excludes>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
在Gradle中，你可以使用标准任务的DSL（领域特定语言）特性创建一个新的JAR存档，然后在`bootRepackage`任务中使用`withJarTask`属性添加对它的依赖：
```gradle
jar {
    baseName = 'spring-boot-sample-profile'
    version =  '0.0.0'
    excludes = ['**/application.yml']
}

task('execJar', type:Jar, dependsOn: 'jar') {
    baseName = 'spring-boot-sample-profile'
    version =  '0.0.0'
    classifier = 'exec'
    from sourceSets.main.output
}

bootRepackage  {
    withJarTask = tasks['execJar']
}
```

### 80.8 远程调试使用Maven启动的Spring Boot项目

想要为使用Maven启动的Spring Boot应用添加一个远程调试器，你可以使用[mave插件](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/)的jvmArguments属性，详情参考[示例](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/maven-plugin/examples/run-debug.html)。

### 80.9 远程调试使用Gradle启动的Spring Boot项目

想要为使用Gradle启动的Spring Boot应用添加一个远程调试器，你可以使用`build.gradle`的`applicationDefaultJvmArgs`属性或`--debug-jvm`命令行选项。

build.gradle：
```gradle
applicationDefaultJvmArgs = [
    "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005"
]
```
命令行：
```shell
$ gradle run --debug-jvm
```
详情查看[Gradle应用插件](http://www.gradle.org/docs/current/userguide/application_plugin.html)。

### 81. 传统部署

### 81.1 创建可部署的war文件

产生一个可部署war包的第一步是提供一个`SpringBootServletInitializer`子类，并覆盖它的`configure`方法，这充分利用了Spring框架对Servlet 3.0的支持，并允许你在应用通过servlet容器启动时配置它。通常，你只需把应用的主类改为继承`SpringBootServletInitializer`即可：
```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
```
下一步是更新你的构建配置，这样你的项目将产生一个war包而不是jar包。如果你使用Maven，并使用`spring-boot-starter-parent`（为了配置Maven的war插件），所有你需要做的就是更改`pom.xml`的打包方式为`war`：
```xml
<packaging>war</packaging>
```
如果你使用Gradle，你需要修改`build.gradle`来将war插件应用到项目上：
```gradle
apply plugin: 'war'
```
该过程最后的一步是确保内嵌的servlet容器不能干扰war包将部署的servlet容器。为了达到这个目的，你需要将内嵌容器的依赖标记为`provided`。

如果使用Maven：
```xml
<dependencies>
    <!-- … -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- … -->
</dependencies>
```
如果使用Gradle：
```gradle
dependencies {
    // …
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
    // …
}
```
如果你使用[Spring Boot构建工具](../VIII. Build tool plugins/README.md)，将内嵌容器依赖标记为`provided`将产生一个可执行war包，在`lib-provided`目录有该war包的`provided`依赖。这意味着，除了部署到servlet容器，你还可以通过使用命令行`java -jar`命令来运行应用。

**注** 查看Spring Boot基于以上配置的一个[Maven示例应用](http://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples/spring-boot-sample-traditional/pom.xml)。

### 81.2 为老的servlet容器创建可部署的war文件

老的Servlet容器不支持在Servlet 3.0中使用的`ServletContextInitializer`启动处理。你仍旧可以在这些容器使用Spring和Spring Boot，但你需要为应用添加一个`web.xml`，并将它配置为通过一个`DispatcherServlet`加载一个`ApplicationContext`。

### 81.3 将现有的应用转换为Spring Boot

对于一个非web项目，转换为Spring Boot应用很容易（抛弃创建`ApplicationContext`的代码，取而代之的是调用`SpringApplication`或`SpringApplicationBuilder`）。Spring MVC web应用通常先创建一个可部署的war应用，然后将它迁移为一个可执行的war或jar，建议阅读[Getting Started Guide on Converting a jar to a war.](http://spring.io/guides/gs/convert-jar-to-war/)。

通过继承`SpringBootServletInitializer`创建一个可执行war（比如，在一个名为`Application`的类中），然后添加Spring Boot的`@EnableAutoConfiguration`注解，示例：
```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        // Customize the application or call application.sources(...) to add sources
        // Since our example is itself a @Configuration class we actually don't
        // need to override this method.
        return application;
    }

}
```
记住不管你往`sources`放什么东西，它仅是一个Spring `ApplicationContext`，正常情况下，任何生效的在这里也会起作用。有一些beans你可以先移除，然后让Spring Boot提供它的默认实现，不过有可能需要先完成一些事情。

静态资源可以移到classpath根目录下的`/public`（或`/static`，`/resources`，`/META-INF/resources`）。同样的方式也适合于`messages.properties`（Spring Boot在classpath根目录下自动发现这些配置）。

美妙的（Vanilla usage of）Spring `DispatcherServlet`和Spring Security不需要改变。如果你的应用有其他特性，比如使用其他servlets或filters，那你可能需要添加一些配置到你的`Application`上下文中，按以下操作替换`web.xml`的那些元素：

- 在容器中安装一个`Servlet`或`ServletRegistrationBean`类型的`@Bean`，就好像`web.xml`中的`<servlet/>`和`<servlet-mapping/>`。
- 同样的添加一个`Filter`或`FilterRegistrationBean`类型的`@Bean`（类似于`<filter/>`和`<filter-mapping/>`）。
- 在XML文件中的`ApplicationContext`可以通过`@Import`添加到你的`Application`中。简单的情况下，大量使用注解配置可以在几行内定义`@Bean`定义。

一旦war可以使用，我们就通过添加一个main方法到`Application`来让它可以执行，比如：
```java
public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
}
```
应用可以划分为多个类别：

- 没有web.xml的Servlet 3.0+应用
- 有web.xml的应用
- 有上下文层次的应用
- 没有上下文层次的应用

所有这些都可以进行适当的转化，但每个可能需要稍微不同的技巧。

Servlet 3.0+的应用转化的相当简单，如果它们已经使用Spring Servlet 3.0+初始化器辅助类。通常所有来自一个存在的`WebApplicationInitializer`的代码可以移到一个`SpringBootServletInitializer`中。如果一个存在的应用有多个`ApplicationContext`（比如，如果它使用`AbstractDispatcherServletInitializer`），那你可以将所有上下文源放进一个单一的`SpringApplication`。你遇到的主要难题可能是如果那样不能工作，那你就要维护上下文层次。参考示例[entry on building a hierarchy](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-build-an-application-context-hierarchy)。一个存在的包含web相关特性的父上下文通常需要分解，这样所有的`ServletContextAware`组件都处于子上下文中。

对于还不是Spring应用的应用来说，上面的指南有助于你把应用转换为一个Spring Boot应用，但你也可以选择其他方式。

### 81.4 部署WAR到Weblogic

想要将Spring Boot应用部署到Weblogic，你需要确保你的servlet初始化器直接实现`WebApplicationInitializer`（即使你继承的基类已经实现了它）。

一个传统的Weblogic初始化器可能如下所示：
```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.web.SpringBootServletInitializer;
import org.springframework.web.WebApplicationInitializer;

@SpringBootApplication
public class MyApplication extends SpringBootServletInitializer implements WebApplicationInitializer {

}
```
如果使用logback，你需要告诉Weblogic你倾向使用的打包版本而不是服务器预装的版本。你可以通过添加一个具有如下内容的`WEB-INF/weblogic.xml`实现该操作：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<wls:weblogic-web-app
	xmlns:wls="http://xmlns.oracle.com/weblogic/weblogic-web-app"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
		http://java.sun.com/xml/ns/javaee/ejb-jar_3_0.xsd
		http://xmlns.oracle.com/weblogic/weblogic-web-app
		http://xmlns.oracle.com/weblogic/weblogic-web-app/1.4/weblogic-web-app.xsd">
	<wls:container-descriptor>
		<wls:prefer-application-packages>
			<wls:package-name>org.slf4j</wls:package-name>
		</wls:prefer-application-packages>
	</wls:container-descriptor>
</wls:weblogic-web-app>
```

### 81.5 部署WAR到老的(Servlet2.5)容器

Spring Boot使用 Servlet 3.0 APIs初始化`ServletContext`（注册`Servlets`等），所以你不能在一个Servlet 2.5的容器中原封不动的使用同样的应用。使用一些特定的工具也是可以在老的容器中运行Spring Boot应用的。如果添加了`org.springframework.boot:spring-boot-legacy`依赖，你只需要创建一个`web.xml`，声明一个用于创建应用上下文的上下文监听器，过滤器和servlets。上下文监听器是专用于Spring Boot的，其他的都是一个Servlet 2.5的Spring应用所具有的。示例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>demo.Application</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.boot.legacy.context.web.SpringBootContextLoaderListener</listener-class>
    </listener>

    <filter>
        <filter-name>metricFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>metricFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextAttribute</param-name>
            <param-value>org.springframework.web.context.WebApplicationContext.ROOT</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>appServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```
在该示例中，我们使用一个单一的应用上下文（通过上下文监听器创建的），然后使用一个init参数将它附加到`DispatcherServlet`。这在一个Spring Boot应用中是很正常的（你通常只有一个应用上下文）。
