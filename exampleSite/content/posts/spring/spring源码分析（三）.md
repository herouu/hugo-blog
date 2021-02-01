---
title: spring源码分析（三）：说说prepareRefresh(),obtainFreshBeanFactory(),prepareBeanFactory(beanFactory)
date: 2019-01-16 20:12:35
tags: ["spring源码"]

---

&emsp;&emsp;无论 spring 基于何种配置，最后都会执行 AbstractApplicationContext 下的 refresh 方法，今天说说下面这三行代码

```java
// Prepare this context for refreshing.
			// 准备上下文环境
			prepareRefresh();
			// Tell the subclass to refresh the internal bean factory.
			// 初始化BeanFactory,从配置文件.xml中读取bean,bean在此处进行实例化
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			// Prepare the bean factory for use in this context.
			// 对BeanFactory进行填充，设置BeanFactory的一些属性
			prepareBeanFactory(beanFactory);
```

虽说只有三行代码，牵扯的东西还是比较多的

<!--more-->

### prepareRefresh()方法

&emsp;&emsp;这个方法主要是准备 spring 的上下文环境,看一下方法的实现

```java
protected void prepareRefresh() {
  this.startupDate = System.currentTimeMillis();
  this.closed.set(false);
  this.active.set(true);
  if (logger.isDebugEnabled()) {
    if (logger.isTraceEnabled()) {
      logger.trace("Refreshing " + this);
    }
    else {
      logger.debug("Refreshing " + getDisplayName());
    }
  }

  // Initialize any placeholder property sources in the context environment
  // 读取环境变量，主要是配置，初始换环境变量的工作是在加载配置文件的时候
  initPropertySources();
  // Validate that all properties marked as required are resolvable
  // see ConfigurablePropertyResolver#setRequiredProperties
  // 验证所有的配置是否能够被解析，即是否满足k->v的数据结构
  getEnvironment().validateRequiredProperties();
  // Allow for the collection of early ApplicationEvents,
  // to be published once the multicaster is available...
  this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### initPropertySources();方法

&emsp;&emsp;抽象类默认`default do nothing`，由其子类进行实现

```java
protected void initPropertySources() {
  // For subclasses: do nothing by default.
}
```

&emsp;&emsp;其中一个子类的实现

```java
@Override
protected void initPropertySources() {
  ConfigurableEnvironment env = getEnvironment();
  if (env instanceof ConfigurableWebEnvironment) {
    ((ConfigurableWebEnvironment) env).initPropertySources(this.servletContext, null);
  }
}
```

这里的 getEnvironment();在此篇[spring 源码分析（一）](/spring源码分析（一）)分析过，这里就不赘述。

#### validateRequiredProperties()方法

&emsp;&emsp;这个方法主要就是验证数据是否满足 k-v 的数据结构，并是否能获取到 key 的值，如果获取不到，则抛出`MissingRequiredPropertiesException`异常。

```java
@Override
public void validateRequiredProperties() {
  MissingRequiredPropertiesException ex = new MissingRequiredPropertiesException();
  for (String key : this.requiredProperties) {
    if (this.getProperty(key) == null) {
      ex.addMissingRequiredProperty(key);
    }
  }
  if (!ex.getMissingRequiredProperties().isEmpty()) {
    throw ex;
  }
}
```

### obtainFreshBeanFactory()方法

&emsp;&emsp;这个方法的实现

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
  refreshBeanFactory();
  return getBeanFactory();
}
```

#### refreshBeanFactory()方法

&emsp;&emsp;这个方法做了以下几件事情

- 如果 BeanFactory 已经存在，销毁 BeanFactory 并清空一些属性。
- 初始化 BeanFactory
- 如果有定制化的 BeanFactory 则加载 customizeBeanFactory(beanFactory)
- 读取\*.xml 中的 bean loadBeanDefinitions(beanFactory) ，比如在 spring.xml 中添加注解扫描配置，在这个方法中，进行 xml 文件解析并进行 bean 的实例化,比如：@Service、@Controller、@Component 等等基于注解配置的 Bean。loadBeanDefinitions(beanFactory)，会在 [spring 源码分析(四)]() 进行详细说明.

```xml
  <context:component-scan base-package="org.springframework.context.annotation"
          use-default-filters="false"
          annotation-config="false">
      <context:include-filter type="assignable"
              expression="org.springframework.context.annotation.ComponentScanParserBeanDefinitionDefaultsTests$DefaultsTestBean"/>
  </context:component-scan>
```

&emsp;&emsp;这是 refreshBeanFactory()方法的实现

```Java
@Override
protected final void refreshBeanFactory() throws BeansException {
  if (hasBeanFactory()) {
    destroyBeans();
    closeBeanFactory();
  }
  try {
    DefaultListableBeanFactory beanFactory = createBeanFactory();
    beanFactory.setSerializationId(getId());
    // 定制化的BeanFactory
    customizeBeanFactory(beanFactory);
    // 读取*xml文件中的bean
    loadBeanDefinitions(beanFactory);
    synchronized (this.beanFactoryMonitor) {
      this.beanFactory = beanFactory;
    }
  }
  catch (IOException ex) {
    throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
  }
}
```

### prepareBeanFactory(beanFactory)方法

&emsp;&emsp;prepareBeanFactory(beanFactory)方法的实现

```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
  // Tell the internal bean factory to use the context's class loader etc.
  // 设置bean的类加载器，bean表达式解析器，可编辑属性的注册器
  beanFactory.setBeanClassLoader(getClassLoader());
  beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
  beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

  // Configure the bean factory with context callbacks.
  // 配置一些回调接口，设置后置处理器，忽略一些接口的依赖，添加注解处理器
  beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
  beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
  beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
  beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
  beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
  beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
  beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

  // BeanFactory interface not registered as resolvable type in a plain factory.
  // MessageSource registered (and found for autowiring) as a bean.
  beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
  beanFactory.registerResolvableDependency(ResourceLoader.class, this);
  beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
  beanFactory.registerResolvableDependency(ApplicationContext.class, this);

  // Register early post-processor for detecting inner beans as ApplicationListeners.
  // 添加监听器
  beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

  // Detect a LoadTimeWeaver and prepare for weaving, if found.
  // 添加切面处理器
  if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
    beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
    // Set a temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
  }

  // Register default environment beans.
  // 注册默认的bean对象
  if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
    beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
  }
  if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
    beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
  }
  if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
    beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
  }
}
```
