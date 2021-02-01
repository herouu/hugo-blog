---
title: Spring源码分析（一）：如何阅读spring源码
date: 2018-12-31 16:28:52
tags: ["spring源码"]

---

&emsp;&emsp;使用 spring 框架已经很久了，spring 框架的 IoC 和 AOP 可以说是 Spring 框架的核心技术，作为一个 java 开发人员，有必要去了解一下源码层面的东西，倒不是说，非要根据源码来重复造轮子。而是在使用轮子的时候，清楚轮子的使用时机和轮子的实现方式。做软件开发，有的人知其然，不知其所以然，能干活；有的人，知其然，知其所以然，能干活。大概后者，干活能快点？

<!--more-->

### Spring 的入口

&emsp;&emsp;记得自己刚接触 spring 的时候，使用最多还是 Servlet3.0 版本,现在 Servlet 都干到 4.0.1 了，不禁感叹，岁月真是一把杀猪刀，恍惚之间，好几年过去了，跑偏了。这里为什么会说到 Servlet？现在做项目的时候可能很少用 servlet+jsp 这样的技术组合，用这样的技术组合可能是上一代 java 开发的故事了，但是 servlet 不重要了吗？不，很重要。关于 Servlet 是什么？请参考[维基百科](https://zh.wikipedia.org/wiki/Java_Servlet)，通俗的解释就是，web 应用基于 HTTP 协议，而 servlet 是 sun 公司提供一套规范（接口），用来处理客户端请求、响应给浏览器的动态资源，而接口的实现交给容器（tomcat,jetty...）。只是现在开发的 java web 应用都是基于 spring,spring 对 http 请求的发送以及响应封装的太好了，以至于 servlet 这种东西已然成了底层的底层：
`spring->spring源码->servlet->HTTP/HTTP2`层级关系大概是这样的。
&emsp;&emsp;如果使用的是 tomcat，tomcat 实现了 servlet 规范，在启动 tomcat 的时候，初始化资源，调用实现 ServletContextListener 接口的对象，spring 框架的初始化即由此时开始。
![tomcat启动，加载web.xml](https://bj.bcebos.com/v1/alertcode-blog/Spring源码分析（一）：如何阅读spring源码/tomcat容器初始化加载ContextLoaderListener.jpg)
&emsp;&emsp;在 ContextLoaderListener 实现的 contextInitialized 方法中调用的 initWebApplicationContext 方法是一切的开始。

```Java
@Override
public void contextInitialized(ServletContextEvent event) {
  //所有的一切从此开始
  initWebApplicationContext(event.getServletContext());
}
```

### 怎么看 spring 框架的源码

&emsp;&emsp;既然知道 ContextLoaderListener 是一切的入口，那自然从入口处开始 debug 是再好不过的选择，但是我们要新建一个 spring 工程，配置 web.xml 吗，当然不用。我 debug 的方式是，直接下载 git clone spring 的源码工程，按照 import-into-idea.md 文件描述的步骤，将工程导入 idea,去掉`spring-aspects`工程，如下图所示：
![去掉spring-aspects工程后编译](https://bj.bcebos.com/v1/alertcode-blog/Spring源码分析（一）：如何阅读spring源码/spring源码解析_20181231222538.png)
&emsp;&emsp;找到 org.springframework.web.context.ContextLoaderTests 单元测试，直接对 testContextLoaderListenerWithDefaultContext 方法进行 debug,看源码原来如此简单 😄😄😏
![junit开始源码阅读](https://bj.bcebos.com/v1/alertcode-blog/Spring源码分析（一）：如何阅读spring源码/spring-1-源码阅读的开始_20181231223217.png)

### ContextLoader.initWebApplicationContext 方法

&emsp;&emsp;断点进入该方法 initWebApplicationContext 下的 configureAndRefreshWebApplicationContext 方法

```java
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
  if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
    throw new IllegalStateException(
        "Cannot initialize context because there is already a root application context present - " +
        "check whether you have multiple ContextLoader* definitions in your web.xml!");
  }

  servletContext.log("Initializing Spring root WebApplicationContext");
  Log logger = LogFactory.getLog(ContextLoader.class);
  if (logger.isInfoEnabled()) {
    logger.info("Root WebApplicationContext: initialization started");
  }
  long startTime = System.currentTimeMillis();

  try {
    // Store context in local instance variable, to guarantee that
    // it is available on ServletContext shutdown.
    if (this.context == null) {
      this.context = createWebApplicationContext(servletContext);
    }
    if (this.context instanceof ConfigurableWebApplicationContext) {
      ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
      if (!cwac.isActive()) {
        // The context has not yet been refreshed -> provide services such as
        // setting the parent context, setting the application context id, etc
        if (cwac.getParent() == null) {
          // The context instance was injected without an explicit parent ->
          // determine parent for root web application context, if any.
          ApplicationContext parent = loadParentContext(servletContext);
          cwac.setParent(parent);
        }
        // 配置并刷新WebApplicationContext,这个位置断点进入
        configureAndRefreshWebApplicationContext(cwac, servletContext);
      }
    }
    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

    ClassLoader ccl = Thread.currentThread().getContextClassLoader();
    if (ccl == ContextLoader.class.getClassLoader()) {
      currentContext = this.context;
    }
    else if (ccl != null) {
      currentContextPerThread.put(ccl, this.context);
    }

    if (logger.isInfoEnabled()) {
      long elapsedTime = System.currentTimeMillis() - startTime;
      logger.info("Root WebApplicationContext initialized in " + elapsedTime + " ms");
    }

    return this.context;
  }
  catch (RuntimeException | Error ex) {
    logger.error("Context initialization failed", ex);
    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
    throw ex;
  }
}
```

### 关注 configureAndRefreshWebApplicationContext 下 wac.refresh()

```java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
  if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
    // The application context id is still set to its original default value
    // -> assign a more useful id based on available information
    String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
    if (idParam != null) {
      wac.setId(idParam);
    }
    else {
      // Generate default id...
      wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
          ObjectUtils.getDisplayString(sc.getContextPath()));
    }
  }

  wac.setServletContext(sc);
  String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);
  if (configLocationParam != null) {
    wac.setConfigLocation(configLocationParam);
  }

  // The wac environment's #initPropertySources will be called in any case when the context
  // is refreshed; do it eagerly here to ensure servlet property sources are in place for
  // use in any post-processing or initialization that occurs below prior to #refresh
  // 这里进行环境的获取
  ConfigurableEnvironment env = wac.getEnvironment();
  if (env instanceof ConfigurableWebEnvironment) {
    ((ConfigurableWebEnvironment) env).initPropertySources(sc, null);
  }

  customizeContext(sc, wac);
  // 这个方法比较重要对WebApplicationContext对象进行刷新
  // bean对象的初始化，注入都是在这个方法中进行的
  wac.refresh();
}
```

#### 来看一下 wac.getEnvironment()这个方法

&emsp;&emsp;看下它的实现

```java
@Override
// 如果没有初始环境变量，则创建环境变量，如果环境变量存在，则返回
public ConfigurableEnvironment getEnvironment() {
  if (this.environment == null) {
    // 初始化环境变量是org.springframework.web.context.ContextLoader.initWebApplicationContext调用
    // org.springframework.web.context.ContextLoader.configureAndRefreshWebApplicationContext方法的
    // 时候进行环境变量的初始化，configureAndRefreshWebApplicationContext主要是加载xml配置文件，
    // 在解析配置文件的时候，resolvePath 方法中会调用 getEnvironment 这个时候环境变量没有初始化
    // 会初始化一个StandardEnvironment对象
    this.environment = createEnvironment();
  }
  return this.environment;
}
```

&emsp;&emsp;createEnvironment()做了什么?

```java
// new 一个默认的标准的环境变量
// ConfigurableEnvironment接口只有一个StandardEnvironment实现，
// StandardServletEnvironment继承自StandardEnvironment
protected ConfigurableEnvironment createEnvironment() {
  return new StandardEnvironment();
}
```

### 调用 AbstractApplicationContext.refresh()

&emsp;&emsp;这个方法几乎包含了 ApplicationContext 接口定义的所有方法，且每一个方法都有很多旁支末节，Spring Boot 启动时断点也会进入到此方法中，以后会对这些方法重点关注与分析。

```Java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			// 准备刷新上下文环境
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			// 初始化BeanFactory
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			// 对BeanFactory进行填充
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 后处理器初始化
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				// 调用后置处理器
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				// 注册拦截Bean处理器，这里只是注册，真正的调用是在getBean的时候
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				// 初始化消息源，国际化处理
				initMessageSource();

				// Initialize event multicaster for this context.
				// 初始化消息广播器，并放入"applicationEventMulticaster" bean中
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				// 刷新
				onRefresh();

				// Check for listener beans and register them.
				// 注册监听器
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				// 完成剩下的（non-lazy-init）BeanFactory初始化
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				// 完成刷新过程
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				// 销毁已经创建的Bean
				destroyBeans();

				// Reset 'active' flag.
				// 取消刷新，active 将active属性设置为false
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				// 重置缓存
				resetCommonCaches();
			}
		}
	}
```
