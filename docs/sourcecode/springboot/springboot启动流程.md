## 构造方法

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
   this.resourceLoader = resourceLoader; //资源加载器
   Assert.notNull(primarySources, "PrimarySources must not be null");
   this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources)); //主要的 Java Config 类的数组
   this.webApplicationType = WebApplicationType.deduceFromClasspath(); // Web 应用类型
   this.bootstrapRegistryInitializers = new ArrayList<>(
         getSpringFactoriesInstances(BootstrapRegistryInitializer.class));  
   // 初始化 initializers 属性
   setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
   // 初始化 listeners 属性
   setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
   //获得是调用了哪个main(String[] args) 方法
   this.mainApplicationClass = deduceMainApplicationClass();
}
```

### getSpringFactoriesInstances 

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
   return getSpringFactoriesInstances(type, new Class<?>[] {});
}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
   ClassLoader classLoader = getClassLoader();
   // <1> 加载指定类型对应的，在 `META-INF/spring.factories` 里的类名的数组
   Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
   // <2> 创建对象们 
   List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
   // <3> 排序对象们,例如说，类上有 @Order 注解。
   AnnotationAwareOrderComparator.sort(instances);
   return instances;
}
```

#### createSpringFactoriesInstances

```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
      ClassLoader classLoader, Object[] args, Set<String> names) {
   // 遍历 names 数组
   List<T> instances = new ArrayList<>(names.size());
   for (String name : names) {
      try {
         // 获得 name 对应的类
         Class<?> instanceClass = ClassUtils.forName(name, classLoader);
         // 判断类是否实现自 type 类
         Assert.isAssignable(type, instanceClass);
         // 获得构造方法
         Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
         // 创建对象
         T instance = (T) BeanUtils.instantiateClass(constructor, args);
         instances.add(instance);
      }
      catch (Throwable ex) {
         throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
      }
   }
   return instances;
}
```



## run

```java
public ConfigurableApplicationContext run(String... args) {
   // <1> 用于简单统计 run 启动过程的时长
   long startTime = System.nanoTime();
   DefaultBootstrapContext bootstrapContext = createBootstrapContext();
   ConfigurableApplicationContext context = null;
   // 配置 headless 属性
   configureHeadlessProperty();
   // <2> 获得 SpringApplicationRunListener 的数组，并启动监听
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting(bootstrapContext, this.mainApplicationClass);
   try {
     	// <3> 创建  ApplicationArguments 对象
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
     	// <4> 加载属性配置。执行完成后，所有的 environment 的属性都会加载进来，包括 application.properties 和外部的属性配置。
      ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
      configureIgnoreBeanInfo(environment);
     	// <5> 打印 Spring Banner
      Banner printedBanner = printBanner(environment);
      // <6> 创建 Spring 容器。
      context = createApplicationContext();
      context.setApplicationStartup(this.applicationStartup);
      // <8> 主要是调用所有初始化类的 initialize 方法
      prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
      // <9> 启动（刷新） Spring 容器
      refreshContext(context);
      // <10> 执行 Spring 容器的初始化的后置逻辑。默认实现为空。
      afterRefresh(context, applicationArguments);
      // <11> 统计时长
      Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
      // <12> 打印 Spring Boot 启动的时长日志。
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
      }
      // <13> 通知 SpringApplicationRunListener 的数组，Spring 容器启动完成
      listeners.started(context, timeTakenToStartup);
      // <14> 调用 ApplicationRunner 或者 CommandLineRunner 的运行方法。
      callRunners(context, applicationArguments);
   }
   catch (Throwable ex) { 
      // <14.1> 如果发生异常，则进行处理，并抛出 IllegalStateException 异常
      handleRunFailure(context, ex, listeners);
      throw new IllegalStateException(ex);
   }
   try {
      Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
     	// 通知 SpringApplicationRunListener 的数组，Spring 容器运行中
      listeners.ready(context, timeTakenToReady);
   }
   catch (Throwable ex) {
      handleRunFailure(context, ex, null);
      throw new IllegalStateException(ex);
   }
   return context;
}
```



#### getRunListeners

```java
// 获得 SpringApplicationRunListener 数组
private SpringApplicationRunListeners getRunListeners(String[] args) {
   Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
   return new SpringApplicationRunListeners(logger,
         getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args),
         this.applicationStartup);
}
```



#### prepareEnvironment

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
      DefaultBootstrapContext bootstrapContext, ApplicationArguments applicationArguments) {
   // <1> 创建 ConfigurableEnvironment 对象，并进行配置
   ConfigurableEnvironment environment = getOrCreateEnvironment();
   configureEnvironment(environment, applicationArguments.getSourceArgs());
   ConfigurationPropertySources.attach(environment);
   // <2> 通知 SpringApplicationRunListener 的数组，环境变量已经准备完成。
   listeners.environmentPrepared(bootstrapContext, environment);
   DefaultPropertiesPropertySource.moveToEnd(environment);
   Assert.state(!environment.containsProperty("spring.main.environment-prefix"),
         "Environment prefix cannot be set via properties.");
   // <3> 绑定 environment 到 SpringApplication 上
   bindToSpringApplication(environment);
   if (!this.isCustomEnvironment) {
      environment = convertEnvironment(environment);
   }
   // <5> 如果有 attach 到 environment 上的 MutablePropertySources ，则添加到 environment 的 PropertySource 中。
   ConfigurationPropertySources.attach(environment);
   return environment;
}
```

##### getOrCreateEnvironment

```java
private ConfigurableEnvironment getOrCreateEnvironment() {
   // 已经存在，则进行返回
   if (this.environment != null) {
      return this.environment;
   }
   // 不存在，则根据 webApplicationType 类型，进行创建。
   switch (this.webApplicationType) {
   case SERVLET:
      return new ApplicationServletEnvironment();
   case REACTIVE:
      return new ApplicationReactiveWebEnvironment();
   default:
      return new ApplicationEnvironment();
   }
}
```

##### configureEnvironment

```java
protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
   // <1.1> 设置 environment 的 conversionService 属性
   if (this.addConversionService) {
      environment.setConversionService(new ApplicationConversionService());
   }
   // <1.2> 增加 environment 的 PropertySource 属性源
   configurePropertySources(environment, args);
   // <1.3> 配置 environment 的 activeProfiles 属性
   configureProfiles(environment, args);
}
```

###### configurePropertySources

```java
protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
   MutablePropertySources sources = environment.getPropertySources();
   // 配置的 defaultProperties
   if (!CollectionUtils.isEmpty(this.defaultProperties)) {
      DefaultPropertiesPropertySource.addOrMerge(this.defaultProperties, sources);
   }
   // 来自启动参数的
   if (this.addCommandLineProperties && args.length > 0) {
      String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
      // 已存在，就进行替换
      if (sources.contains(name)) {
         PropertySource<?> source = sources.get(name);
         CompositePropertySource composite = new CompositePropertySource(name);
         composite.addPropertySource(
               new SimpleCommandLinePropertySource("springApplicationCommandLineArgs", args));
         composite.addPropertySource(source);
         sources.replace(name, composite);
      }
      else {
         // 不存在，就进行添加
         sources.addFirst(new SimpleCommandLinePropertySource(args));
      }
   }
}
```



#### createApplicationContext

- 根据 `webApplicationType` 类型，获得对应的 ApplicationContext 对象。



#### prepareContext

```java
private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context,
      ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
      ApplicationArguments applicationArguments, Banner printedBanner) {
   // <1> 设置 context 的 environment 属性
   context.setEnvironment(environment);
   // <2> 设置 context 的一些属性
   postProcessApplicationContext(context);
   // <3> 初始化 ApplicationContextInitializer
   applyInitializers(context);
   // <4> 通知 SpringApplicationRunListener 的数组，Spring 容器准备完成。
   listeners.contextPrepared(context);
   bootstrapContext.close(context);
   // <5> 打印日志
   if (this.logStartupInfo) {
      logStartupInfo(context.getParent() == null);
      logStartupProfileInfo(context);
   }
   // <6> 设置 beanFactory 的属性
   ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
   beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
   if (printedBanner != null) {
      beanFactory.registerSingleton("springBootBanner", printedBanner);
   }
   if (beanFactory instanceof AbstractAutowireCapableBeanFactory) {
      ((AbstractAutowireCapableBeanFactory) beanFactory).setAllowCircularReferences(this.allowCircularReferences);
      if (beanFactory instanceof DefaultListableBeanFactory) {
         ((DefaultListableBeanFactory) beanFactory)
               .setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
      }
   }
   if (this.lazyInitialization) {
      context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
   }
   context.addBeanFactoryPostProcessor(new PropertySourceOrderingBeanFactoryPostProcessor(context));
   // <7> 加载 BeanDefinition 们
   Set<Object> sources = getAllSources();
   Assert.notEmpty(sources, "Sources must not be empty");
   load(context, sources.toArray(new Object[0]));
  // <8> 通知 SpringApplicationRunListener 的数组，Spring 容器加载完成。
   listeners.contextLoaded(context);
}
```



##### postProcessApplicationContext

```java
// 设置 context 的一些属性
protected void postProcessApplicationContext(ConfigurableApplicationContext context) {
   if (this.beanNameGenerator != null) {
      context.getBeanFactory().registerSingleton(AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR,
            this.beanNameGenerator);
   }
   if (this.resourceLoader != null) {
      if (context instanceof GenericApplicationContext) {
         ((GenericApplicationContext) context).setResourceLoader(this.resourceLoader);
      }
      if (context instanceof DefaultResourceLoader) {
         ((DefaultResourceLoader) context).setClassLoader(this.resourceLoader.getClassLoader());
      }
   }
   if (this.addConversionService) {
      context.getBeanFactory().setConversionService(context.getEnvironment().getConversionService());
   }
}
```



##### applyInitializers

```java
protected void applyInitializers(ConfigurableApplicationContext context) {
   // 遍历 ApplicationContextInitializer 数组
   for (ApplicationContextInitializer initializer : getInitializers()) { 
     // 校验 ApplicationContextInitializer 的泛型非空
      Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(),
            ApplicationContextInitializer.class);
      Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
      // 初始化 ApplicationContextInitializer
      initializer.initialize(context);
   }
}
```



##### Load

```java
// 加载 BeanDefinition 们。
protected void load(ApplicationContext context, Object[] sources) {
   if (logger.isDebugEnabled()) {
      logger.debug("Loading source " + StringUtils.arrayToCommaDelimitedString(sources));
   }
   // <1> 创建 BeanDefinitionLoader 对象
   BeanDefinitionLoader loader = createBeanDefinitionLoader(getBeanDefinitionRegistry(context), sources);
   // <2> 设置 loader 的属性
   if (this.beanNameGenerator != null) {
      loader.setBeanNameGenerator(this.beanNameGenerator);
   }
   if (this.resourceLoader != null) {
      loader.setResourceLoader(this.resourceLoader);
   }
   if (this.environment != null) {
      loader.setEnvironment(this.environment);
   }
   // <3> 执行 BeanDefinition 加载
   loader.load();
}
```



###### getBeanDefinitionRegistry

```java
// 创建 BeanDefinitionRegistry 对象
private BeanDefinitionRegistry getBeanDefinitionRegistry(ApplicationContext context) {
   if (context instanceof BeanDefinitionRegistry) {
      return (BeanDefinitionRegistry) context;
   }
   if (context instanceof AbstractApplicationContext) {
      return (BeanDefinitionRegistry) ((AbstractApplicationContext) context).getBeanFactory();
   }
   throw new IllegalStateException("Could not locate BeanDefinitionRegistry");
}
```





#### refreshContext

```java
private void refreshContext(ConfigurableApplicationContext context) {
   //是否注册 ShutdownHook 钩子
   if (this.registerShutdownHook) {
      // 注册 ShutdownHook 钩子 主要用于 Spring 应用的关闭时，销毁相应的 Bean 们
      shutdownHook.registerApplicationContext(context);
   }
   //开启（刷新）Spring 容器 可以触发 Spring Boot 的自动配置的功能
   refresh(context);
}
```



####  callRunners

```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
   // <1> 获得所有 Runner 们
   List<Object> runners = new ArrayList<>();
   // <1.1> 获得所有 ApplicationRunner Bean 们
   runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
   // <1.2> 获得所有 CommandLineRunner Bean 们
   runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
   // <1.3> 排序 runners
   AnnotationAwareOrderComparator.sort(runners);
   // <2> 遍历 Runner 数组，执行逻辑
   for (Object runner : new LinkedHashSet<>(runners)) {
      if (runner instanceof ApplicationRunner) {
         callRunner((ApplicationRunner) runner, args);
      }
      if (runner instanceof CommandLineRunner) {
         callRunner((CommandLineRunner) runner, args);
      }
   }
}
```





## SpringApplicationRunListeners

```java
// SpringApplicationRunListener 数组的封装
class SpringApplicationRunListeners {

   private final Log log;

   private final List<SpringApplicationRunListener> listeners;

   private final ApplicationStartup applicationStartup;

   SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners,
         ApplicationStartup applicationStartup) {
      this.log = log;
      this.listeners = new ArrayList<>(listeners);
      this.applicationStartup = applicationStartup;
   }

   void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
      doWithListeners("spring.boot.application.starting", (listener) -> listener.starting(bootstrapContext),
            (step) -> {
               if (mainApplicationClass != null) {
                  step.tag("mainApplicationClass", mainApplicationClass.getName());
               }
            });
   }

   void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
      doWithListeners("spring.boot.application.environment-prepared",
            (listener) -> listener.environmentPrepared(bootstrapContext, environment));
   }

   void contextPrepared(ConfigurableApplicationContext context) {
      doWithListeners("spring.boot.application.context-prepared", (listener) -> listener.contextPrepared(context));
   }

   void contextLoaded(ConfigurableApplicationContext context) {
      doWithListeners("spring.boot.application.context-loaded", (listener) -> listener.contextLoaded(context));
   }

   void started(ConfigurableApplicationContext context, Duration timeTaken) {
      doWithListeners("spring.boot.application.started", (listener) -> listener.started(context, timeTaken));
   }

   void ready(ConfigurableApplicationContext context, Duration timeTaken) {
      doWithListeners("spring.boot.application.ready", (listener) -> listener.ready(context, timeTaken));
   }

   void failed(ConfigurableApplicationContext context, Throwable exception) {
      doWithListeners("spring.boot.application.failed",
            (listener) -> callFailedListener(listener, context, exception), (step) -> {
               step.tag("exception", exception.getClass().toString());
               step.tag("message", exception.getMessage());
            });
   }

   private void callFailedListener(SpringApplicationRunListener listener, ConfigurableApplicationContext context,
         Throwable exception) {
      try {
         listener.failed(context, exception);
      }
      catch (Throwable ex) {
         if (exception == null) {
            ReflectionUtils.rethrowRuntimeException(ex);
         }
         if (this.log.isDebugEnabled()) {
            this.log.error("Error handling failed", ex);
         }
         else {
            String message = ex.getMessage();
            message = (message != null) ? message : "no error message";
            this.log.warn("Error handling failed (" + message + ")");
         }
      }
   }

   private void doWithListeners(String stepName, Consumer<SpringApplicationRunListener> listenerAction) {
      doWithListeners(stepName, listenerAction, null);
   }

   private void doWithListeners(String stepName, Consumer<SpringApplicationRunListener> listenerAction,
         Consumer<StartupStep> stepAction) {
      StartupStep step = this.applicationStartup.start(stepName);
      this.listeners.forEach(listenerAction);
      if (stepAction != null) {
         stepAction.accept(step);
      }
      step.end();
   }

}
```

### SpringApplicationRunListener

```java
// SpringApplication 运行的监听器接口
public interface SpringApplicationRunListener {

   /**
    * Called immediately when the run method has first started. Can be used for very
    * early initialization.
    * @param bootstrapContext the bootstrap context
    */
   default void starting(ConfigurableBootstrapContext bootstrapContext) {
   }

   /**
    * Called once the environment has been prepared, but before the
    * {@link ApplicationContext} has been created.
    * @param bootstrapContext the bootstrap context
    * @param environment the environment
    */
   default void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
         ConfigurableEnvironment environment) {
   }

   /**
    * Called once the {@link ApplicationContext} has been created and prepared, but
    * before sources have been loaded.
    * @param context the application context
    */
   default void contextPrepared(ConfigurableApplicationContext context) {
   }

   /**
    * Called once the application context has been loaded but before it has been
    * refreshed.
    * @param context the application context
    */
   default void contextLoaded(ConfigurableApplicationContext context) {
   }

   /**
    * The context has been refreshed and the application has started but
    * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
    * ApplicationRunners} have not been called.
    * @param context the application context.
    * @param timeTaken the time taken to start the application or {@code null} if unknown
    * @since 2.6.0
    */
   default void started(ConfigurableApplicationContext context, Duration timeTaken) {
      started(context);
   }

   /**
    * The context has been refreshed and the application has started but
    * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
    * ApplicationRunners} have not been called.
    * @param context the application context.
    * @since 2.0.0
    * @deprecated since 2.6.0 for removal in 3.0.0 in favor of
    * {@link #started(ConfigurableApplicationContext, Duration)}
    */
   @Deprecated
   default void started(ConfigurableApplicationContext context) {
   }

   /**
    * Called immediately before the run method finishes, when the application context has
    * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
    * {@link ApplicationRunner ApplicationRunners} have been called.
    * @param context the application context.
    * @param timeTaken the time taken for the application to be ready or {@code null} if
    * unknown
    * @since 2.6.0
    */
   default void ready(ConfigurableApplicationContext context, Duration timeTaken) {
      running(context);
   }

   /**
    * Called immediately before the run method finishes, when the application context has
    * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
    * {@link ApplicationRunner ApplicationRunners} have been called.
    * @param context the application context.
    * @since 2.0.0
    * @deprecated since 2.6.0 for removal in 3.0.0 in favor of
    * {@link #ready(ConfigurableApplicationContext, Duration)}
    */
   @Deprecated
   default void running(ConfigurableApplicationContext context) {
   }

   /**
    * Called when a failure occurs when running the application.
    * @param context the application context or {@code null} if a failure occurred before
    * the context was created
    * @param exception the failure
    * @since 2.0.0
    */
   default void failed(ConfigurableApplicationContext context, Throwable exception) {
   }

}
```



#### EventPublishingRunListener

```java
// 将 SpringApplicationRunListener 监听到的事件，转换成对应的 SpringApplicationEvent 事件，发布到监听器们。
//通过这样的方式，可以很方便的将 SpringApplication 启动的各种事件，方便的修改成对应的 SpringApplicationEvent 事件。这样，我们就可以不需要修改 SpringApplication 的代码。或者说，我们认为 EventPublishingRunListener 是一个“转换器”
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {

   private final SpringApplication application;

   private final String[] args;

   private final SimpleApplicationEventMulticaster initialMulticaster;

   public EventPublishingRunListener(SpringApplication application, String[] args) {
      this.application = application;
      this.args = args;
      this.initialMulticaster = new SimpleApplicationEventMulticaster();
      for (ApplicationListener<?> listener : application.getListeners()) {
         this.initialMulticaster.addApplicationListener(listener);
      }
   }

   @Override
   public int getOrder() {
      return 0;
   }

   @Override
   public void starting(ConfigurableBootstrapContext bootstrapContext) {
      this.initialMulticaster
            .multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
   }

   @Override
   public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
         ConfigurableEnvironment environment) {
      this.initialMulticaster.multicastEvent(
            new ApplicationEnvironmentPreparedEvent(bootstrapContext, this.application, this.args, environment));
   }

   @Override
   public void contextPrepared(ConfigurableApplicationContext context) {
      this.initialMulticaster
            .multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
   }

   @Override
   public void contextLoaded(ConfigurableApplicationContext context) {
      for (ApplicationListener<?> listener : this.application.getListeners()) {
         if (listener instanceof ApplicationContextAware) {
            ((ApplicationContextAware) listener).setApplicationContext(context);
         }
         context.addApplicationListener(listener);
      }
      this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
   }

   @Override
   public void started(ConfigurableApplicationContext context, Duration timeTaken) {
      context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context, timeTaken));
      AvailabilityChangeEvent.publish(context, LivenessState.CORRECT);
   }

   @Override
   public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
      context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context, timeTaken));
      AvailabilityChangeEvent.publish(context, ReadinessState.ACCEPTING_TRAFFIC);
   }

   @Override
   public void failed(ConfigurableApplicationContext context, Throwable exception) {
      ApplicationFailedEvent event = new ApplicationFailedEvent(this.application, this.args, context, exception);
      if (context != null && context.isActive()) {
         // Listeners have been registered to the application context so we should
         // use it at this point if we can
         context.publishEvent(event);
      }
      else {
         // An inactive context may not have a multicaster so we use our multicaster to
         // call all of the context's listeners instead
         if (context instanceof AbstractApplicationContext) {
            for (ApplicationListener<?> listener : ((AbstractApplicationContext) context)
                  .getApplicationListeners()) {
               this.initialMulticaster.addApplicationListener(listener);
            }
         }
         this.initialMulticaster.setErrorHandler(new LoggingErrorHandler());
         this.initialMulticaster.multicastEvent(event);
      }
   }

   private static class LoggingErrorHandler implements ErrorHandler {

      private static final Log logger = LogFactory.getLog(EventPublishingRunListener.class);

      @Override
      public void handleError(Throwable throwable) {
         logger.warn("Error calling ApplicationEventListener", throwable);
      }

   }

}
```