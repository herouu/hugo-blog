---
title: I. Spring Boot Documentation
tags: ["springboot"]
date: 2018-06-15 18:16:40
categories: ["springboot翻译"]
---

### 1. 关于本文档

Spring Boot 参考指南有[html](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/html)，[pdf](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/pdf/spring-boot-reference.pdf)和[epub](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/epub/spring-boot-reference.epub)等形式的文档，你可以从[docs.spring.io/spring-boot/docs/current/reference](http://docs.spring.io/spring-boot/docs/current/reference)获取到最新版本。

对本文档的拷贝，不管是电子版还是打印，在保证包含版权声明，并且不收取任何费用的情况下，你可以自由使用，或分发给其他人。

<!--more-->

### 2. 获取帮助

使用 Spring Boot 遇到麻烦，我们很乐意帮忙！

- 尝试[How-to’s](../IX. ‘How-to’ guides/README.md)－它们为多数常见问题提供解决方案。
- 学习 Spring 基础知识－Spring Boot 是在很多其他 Spring 项目上构建的，查看[spring.io](http://spring.io/)站点可以获取丰富的参考文档。如果你刚开始使用 Spring，可以尝试这些[指导](http://spring.io/guides)中的一个。
- 提问题－我们时刻监控着[stackoverflow.com](http://stackoverflow.com/)上标记为[spring-boot](http://stackoverflow.com/tags/spring-boot)的问题。
- 在[github.com/spring-projects/spring-boot/issues](https://github.com/spring-projects/spring-boot/issues)上报告 Spring Boot 的 bug。

**注**：Spring Boot 的一切都是开源的，包括文档！如果你发现文档有问题，或只是想提高它们的质量，请[参与进来](http://github.com/spring-projects/spring-boot/tree/master)！

### 3. 第一步

如果你想对 Spring Boot 或 Spring 有个整体认识，可以从[这里开始](../II. Getting started/README.md)！

- 从零开始：[概述](../II. Getting started/8. Introducing Spring Boot.md)｜[要求](../II. Getting started/9. System Requirements.md)｜[安装](../II. Getting started/10. Installing Spring Boot.md)
- 教程：[第一部分](../II. Getting started/11. Developing your first Spring Boot application.md)｜[第二部分](../II. Getting started/11.3. Writing the code.md)
- 运行示例：[第一部分](../II. Getting started/11.4. Running the example.md)｜[第二部分](../II. Getting started/11.5. Creating an executable jar.md)

### 4. 使用 Spring Boot

准备好使用 Spring Boot 了？[我们已经为你铺好道路](../III. Using Spring Boot/README.md).

- 构建系统：[Maven](../III. Using Spring Boot/13.2. Maven.md)｜[Gradle](../III. Using Spring Boot/13.3. Gradle.md)｜[Ant](../III. Using Spring Boot/13.4. Ant.md)｜[Starters](../III. Using Spring Boot/13.5. Starters.md)
- 最佳实践：[代码结构](../III. Using Spring Boot/14. Structuring your code.md)｜[@Configuration](../III. Using Spring Boot/15. Configuration classes.md)｜[@EnableAutoConfiguration](../III. Using Spring Boot/16. Auto-configuration.md)｜[Beans 和依赖注入](../III. Using Spring Boot/17. Spring Beans and dependency injection.md)
- 运行代码：[IDE](../III. Using Spring Boot/19.1. Running from an IDE.md)｜[Packaged](../III. Using Spring Boot/19.2. Running as a packaged application.md)｜[Maven](../III. Using Spring Boot/19.3. Using the Maven plugin.md)｜[Gradle](../III. Using Spring Boot/19.4. Using the Gradle plugin.md)
- 应用打包：[产品级 jars](../III. Using Spring Boot/21. Packaging your application for production.md)
- Spring Boot 命令行：[使用 CLI](../VII. Spring Boot CLI/README.md)

### 5. 了解 Spring Boot 特性

想要了解更多 Spring Boot 核心特性的详情？[这就是为你准备的](../IV. Spring Boot features/README.md)！

- 核心特性：[SpringApplication](../IV. Spring Boot features/23. SpringApplication.md)｜[外部化配置](../IV. Spring Boot features/24. Externalized Configuration.md)｜[Profiles](../IV. Spring Boot features/25. Profiles.md)｜[日志](../IV. Spring Boot features/26. Logging.md)
- Web 应用：[MVC](../IV. Spring Boot features/27.1. The ‘Spring Web MVC framework’.md)｜[内嵌容器](../IV. Spring Boot features/27.3 Embedded servlet container support.md)
- 使用数据：[SQL](../IV. Spring Boot features/29. Working with SQL databases.md)｜[NO-SQL](../IV. Spring Boot features/30. Working with NoSQL technologies.md)
- 消息：[概述](../IV. Spring Boot features/32. Messaging.md)｜[JMS](../IV. Spring Boot features/32.1. JMS.md)
- 测试：[概述](../IV. Spring Boot features/40. Testing.md)｜[Boot 应用](../IV. Spring Boot features/40.3 Testing Spring Boot applications.md)｜[工具](../IV. Spring Boot features/40.4 Test utilities.md)
- 扩展：[Auto-configuration](../IV. Spring Boot features/43. Creating your own auto-configuration.md)｜[@Conditions](../IV. Spring Boot features/43.3 Condition annotations.md)

### 6. 迁移到生产环境

当你准备将 Spring Boot 应用发布到生产环境时，我们提供了一些你[可能喜欢的技巧](../V. Spring Boot Actuator/README.md)！

- 管理端点：[概述](../V. Spring Boot Actuator/46. Endpoints.md)｜[自定义](../V. Spring Boot Actuator/46.1 Customizing endpoints.md)
- 连接选项：[HTTP](../V. Spring Boot Actuator/47. Monitoring and management over HTTP.md)｜[JMX](../V. Spring Boot Actuator/48. Monitoring and management over JMX.md)｜[SSH](../V. Spring Boot Actuator/49. Monitoring and management using a remote shell.md)
- 监控：[指标](../V. Spring Boot Actuator/50. Metrics.md)｜[审计](../V. Spring Boot Actuator/51. Auditing.md)｜[追踪](../V. Spring Boot Actuator/52. Tracing.md)｜[进程](../V. Spring Boot Actuator/53. Process monitoring.md)

### 7. 高级主题

最后，我们为高级用户准备了一些主题。

- 部署 Spring Boot 应用：[云部署](../VI. Deploying Spring Boot applications/55. Deploying to the cloud.md) | [操作系统服务](../VI. Deploying Spring Boot applications/56.1 Unix&Linux services.md)
- 构建工具插件：[Maven](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-maven-plugin)｜[Gradle](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#build-tool-plugins-gradle-plugin)
- 附录：[应用属性](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/# #auto-configuration-classes)｜[可执行 Jars](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#executable-jar)
