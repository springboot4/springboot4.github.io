# @SpringBootApplication

`@SpringBootApplication` 注解，它在 `spring-boot-autoconfigure` 模块中。是一个**组合**注解。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited //此注解声明自定义注解，在使用此自定义注解时，如果注解在类上面时，子类会自动继承此注解，否则的话，子类不会继承此注解
@SpringBootConfiguration //继承自 @Configuration 注解，所以两者功能也一致。
@EnableAutoConfiguration //核心注解，用于开启自动配置功能。
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) }) //扫描指定路径下的 Component
public @interface SpringBootApplication {

   @AliasFor(annotation = EnableAutoConfiguration.class)
   Class<?>[] exclude() default {};

   @AliasFor(annotation = EnableAutoConfiguration.class)
   String[] excludeName() default {};

   @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
   String[] scanBasePackages() default {};

   @AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
   Class<?>[] scanBasePackageClasses() default {};

   @AliasFor(annotation = Configuration.class)
   boolean proxyBeanMethods() default true;

}
```

## @EnableAutoConfiguration

```java
@AutoConfigurationPackage //自动配置包，获取主程序类所在的包路径，并将包路径（包括子包）下的所有组件注册到 Spring IOC 容器中
@Import(AutoConfigurationImportSelector.class) //核心,处理 @EnableAutoConfiguration 注解的资源导入
public @interface EnableAutoConfiguration {

   String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

   /**
    * Exclude specific auto-configuration classes such that they will never be applied.
    * @return the classes to exclude
    */
   Class<?>[] exclude() default {};

   /**
    * Exclude specific auto-configuration class names such that they will never be
    * applied.
    * @return the class names to exclude
    * @since 1.3.0
    */
   String[] excludeName() default {};

}
```



# 流程

![](https://minio.pigx.vip/oss/2022/08/5hpNTs.png)

- ① 处，refresh 方法的调用，在SpringApplication 启动时，会调用到该方法。



### getImports

```java
// ConfigurationClassParser#DeferredImportSelectorGroupingHandler.java 
// 返回组定义的导入
public Iterable<Group.Entry> getImports() {
   for (DeferredImportSelectorHolder deferredImport : this.deferredImports) {
      this.group.process(deferredImport.getConfigurationClass().getMetadata(),
            deferredImport.getImportSelector()); // <1>
   }
   return this.group.selectImports(); // <2>
}
```



#### <1>process

```java
// AutoConfigurationGroup#process
@Override
public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
   Assert.state(deferredImportSelector instanceof AutoConfigurationImportSelector,
         () -> String.format("Only %s implementations are supported, got %s",
               AutoConfigurationImportSelector.class.getSimpleName(),
               deferredImportSelector.getClass().getName()));
   // <1> 获得 AutoConfigurationEntry 对象 
   AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector) deferredImportSelector)
         .getAutoConfigurationEntry(getAutoConfigurationMetadata(), annotationMetadata);
   // <2> 添加到 autoConfigurationEntries 中
   this.autoConfigurationEntries.add(autoConfigurationEntry);
   // <3> 添加到 entries 中
   for (String importClassName : autoConfigurationEntry.getConfigurations()) {
      this.entries.putIfAbsent(importClassName, annotationMetadata);
   }
}
```

- `annotationMetadata` 参数，一般来说是被 `@SpringBootApplication` 注解的元数据。因为，`@SpringBootApplication` 组合了 `@EnableAutoConfiguration` 注解。
- `deferredImportSelector` 参数，`@EnableAutoConfiguration` 注解的定义的 `@Import` 的类，即 AutoConfigurationImportSelector 对象。

##### <1.1>getAutoConfigurationEntry

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
      AnnotationMetadata annotationMetadata) {
   // <1> 判断是否开启。如未开启，返回空数组。
   if (!isEnabled(annotationMetadata)) {
      return EMPTY_ENTRY;
   } 
   // <2> 获得注解的属性
   AnnotationAttributes attributes = getAttributes(annotationMetadata);
   // <3> 获得符合条件的配置类的数组
   List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
   // <3.1> 移除重复的配置类
   configurations = removeDuplicates(configurations);
   // <4> 获得需要排除的配置类
   Set<String> exclusions = getExclusions(annotationMetadata, attributes);
   // <4.1> 校验排除的配置类是否合法
   checkExcludedClasses(configurations, exclusions);
   // <4.2> 从 configurations 中，移除需要排除的配置类
   configurations.removeAll(exclusions);
   // <5> 根据条件（Condition），过滤掉不符合条件的配置类
   configurations = filter(configurations, autoConfigurationMetadata);
   // <6> 触发自动配置类引入完成的事件
   fireAutoConfigurationImportEvents(configurations, exclusions);
   // <7> 创建 AutoConfigurationEntry 对象
   return new AutoConfigurationEntry(configurations, exclusions);
}
```

##### <1.1.1>isEnabled

```java
// 判断是否开启自动配置
protected boolean isEnabled(AnnotationMetadata metadata) {
   // 判断 "spring.boot.enableautoconfiguration" 配置判断，是否开启自动配置。
   // 默认情况下（未配置），开启自动配置。
   if (getClass() == AutoConfigurationImportSelector.class) {
      return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
   }
   return true;
}
```

##### <1.1.2>getAttributes

```java
// 获得注解的属性此处 getAnnotationClass().getName() 返回的是 @EnableAutoConfiguration 注解，所以这里返回的注解属性，只能     // 是 exclude 和 excludeName 这两。
protected AnnotationAttributes getAttributes(AnnotationMetadata metadata) {
   String name = getAnnotationClass().getName();
   AnnotationAttributes attributes = AnnotationAttributes.fromMap(metadata.getAnnotationAttributes(name, true));
   Assert.notNull(attributes, () -> "No auto-configuration attributes found. Is " + metadata.getClassName()
         + " annotated with " + ClassUtils.getShortName(name) + "?");
   return attributes;
}
```

##### <1.1.3>getCandidateConfigurations

`#getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes)` 方法，获得符合条件的配置类的数组。

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
   // <1> 加载指定类型 EnableAutoConfiguration 对应的，在 `META-INF/spring.factories` 里的类名的数组
   List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
         getBeanClassLoader());
   // 断言，非空
   Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
         + "are using a custom packaging, make sure that file is correct.");
   return configurations;
}
```

`<1>` 处，调用 `#getSpringFactoriesLoaderFactoryClass()` 方法，获得要从 `META-INF/spring.factories` 加载的指定类型为 EnableAutoConfiguration 类。代码如下：

```java
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
   return EnableAutoConfiguration.class;
}
```

`<1>` 处，调用 `SpringFactoriesLoader#loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader)` 方法，加载指定类型 EnableAutoConfiguration 对应的，在 `META-INF/spring.factories` 里的类名的数组

![](https://minio.pigx.vip/oss/2022/08/1EnBFO.png)

##### <1.1.3.1>removeDuplicates

```java
// 移除重复的配置类
protected final <T> List<T> removeDuplicates(List<T> list) {
   return new ArrayList<>(new LinkedHashSet<>(list));
}
```

##### <1.1.4>getExclusions

```java
// 获得需要排除的配置类
protected Set<String> getExclusions(AnnotationMetadata metadata, AnnotationAttributes attributes) {
   Set<String> excluded = new LinkedHashSet<>();
   // 注解上的 exclude 属性
   excluded.addAll(asList(attributes, "exclude"));
   // 注解上的 excludeName 属性
   excluded.addAll(Arrays.asList(attributes.getStringArray("excludeName")));
   // 配置文件的 spring.autoconfigure.exclude 属性
   excluded.addAll(getExcludeAutoConfigurationsProperty());
   return excluded;
}
```

##### <1.1.4.1>checkExcludedClasses

```java
private void checkExcludedClasses(List<String> configurations, Set<String> exclusions) {
   List<String> invalidExcludes = new ArrayList<>(exclusions.size());
   for (String exclusion : exclusions) {
      // 获得 exclusions 不在 invalidExcludes 的集合，添加到 invalidExcludes 中
      if (ClassUtils.isPresent(exclusion, getClass().getClassLoader()) && !configurations.contains(exclusion)) {
         invalidExcludes.add(exclusion);
      }
   }
   // 如果 invalidExcludes 非空，抛出 IllegalStateException 异常
   if (!invalidExcludes.isEmpty()) {
      handleInvalidExcludes(invalidExcludes);
   }
}

/**
 * Handle any invalid excludes that have been specified.
 * @param invalidExcludes the list of invalid excludes (will always have at least one
 * element)
 */
protected void handleInvalidExcludes(List<String> invalidExcludes) {
   StringBuilder message = new StringBuilder();
   for (String exclude : invalidExcludes) {
      message.append("\t- ").append(exclude).append(String.format("%n"));
   }
   throw new IllegalStateException(String.format(
         "The following classes could not be excluded because they are not auto-configuration classes:%n%s",
         message));
}
```

##### <1.1.5>filter

`#filter(List<String> configurations, AutoConfigurationMetadata autoConfigurationMetadata)` 方法，根据条件（Condition），过滤掉不符合条件的配置类。

##### <1.1.6>fireAutoConfigurationImportEvents

```java
private void fireAutoConfigurationImportEvents(List<String> configurations, Set<String> exclusions) {
   // <1> 加载指定类型 AutoConfigurationImportListener 对应的，在 `META-INF/spring.factories` 里的类名的数组。
   List<AutoConfigurationImportListener> listeners = getAutoConfigurationImportListeners();
   if (!listeners.isEmpty()) {
     	// <2> 创建 AutoConfigurationImportEvent 事件
      AutoConfigurationImportEvent event = new AutoConfigurationImportEvent(this, configurations, exclusions);
     // <3> 遍历 AutoConfigurationImportListener 监听器们，逐个通知
     for (AutoConfigurationImportListener listener : listeners) {
         // <3.1> 设置 AutoConfigurationImportListener 的属性
         invokeAwareMethods(listener);
         // <3.2> 通知监听器
         listener.onAutoConfigurationImportEvent(event);
      }
   }
}
```





#### <2>selectImports

```java
// AutoConfigurationImportSelector#AutoConfigurationGroup.java
// 获得要引入的配置类
@Override
public Iterable<Entry> selectImports() {
   // <1> 如果为空，则返回空数组
   if (this.autoConfigurationEntries.isEmpty()) {
      return Collections.emptyList();
   }
   // <2.1> 获得 allExclusions
   Set<String> allExclusions = this.autoConfigurationEntries.stream()
         .map(AutoConfigurationEntry::getExclusions).flatMap(Collection::stream).collect(Collectors.toSet());
   // <2.2> 获得 processedConfigurations
   Set<String> processedConfigurations = this.autoConfigurationEntries.stream()
         .map(AutoConfigurationEntry::getConfigurations).flatMap(Collection::stream)
         .collect(Collectors.toCollection(LinkedHashSet::new));
   // <2.3> 从 processedConfigurations 中，移除排除的
   processedConfigurations.removeAll(allExclusions);
   // <3> 处理，返回结果
   return sortAutoConfigurations(processedConfigurations, getAutoConfigurationMetadata()).stream()
         .map((importClassName) -> new Entry(this.entries.get(importClassName), importClassName))
         .collect(Collectors.toList());
}
```