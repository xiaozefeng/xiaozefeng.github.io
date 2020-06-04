# Spring Boot 自动配置原理

Spring Boot 自动配置原理先从 @SpringBootApplication 这个注解说起，追踪原理发现 @SpringBootApplication 主要由3个注解组成

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

@SpringBootConfiguration 里面其实就是 @Configuration

@ComponentScan 的作用就是扫描组件并加入 Spring 的容器

自动配置的原理还是要继续看 @EnableAutoConfiguration

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```

@EnableAutoConfiguration 上的 @Import 通过导入 AutoConfigurationImportSelector这个类，我们进入看看

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
      
      // 实现了 sleectImports 方法
      	@Override
        public String[] selectImports(AnnotationMetadata annotationMetadata) {
          if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
          }
          AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
              .loadMetadata(this.beanClassLoader);
          AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
              annotationMetadata);
          return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
}
```

层层跟进，最后会调用到这个方法

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
   MultiValueMap<String, String> result = cache.get(classLoader);
   if (result != null) {
      return result;
   }

   try {
     //FACTORIES_RESOURCE_LOCATION 其实就是 META-INF/spring.factories 文件
      Enumeration<URL> urls = (classLoader != null ?
            classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
            ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
      result = new LinkedMultiValueMap<>();
      while (urls.hasMoreElements()) {
         URL url = urls.nextElement();
         UrlResource resource = new UrlResource(url);
         Properties properties = PropertiesLoaderUtils.loadProperties(resource);
         for (Map.Entry<?, ?> entry : properties.entrySet()) {
            String factoryTypeName = ((String) entry.getKey()).trim();
            for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
               result.add(factoryTypeName, factoryImplementationName.trim());
            }
         }
      }
      cache.put(classLoader, result);
      return result;
   }
   catch (IOException ex) {
      throw new IllegalArgumentException("Unable to load factories from location [" +
            FACTORIES_RESOURCE_LOCATION + "]", ex);
   }
```



## 总结

1. Spring Boot 在启动时会扫描项目所依赖的 `jar` 包，寻找含有 `spring.factories`
2. 根据 `spring.factories` 加载配置的 `AutoConfigure`类
3. 根据Conditional注解的条件，进行自动配置并将bean注入到 Spring Context中

