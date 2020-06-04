# Spring Boot 自定义 starter

### 创建 maven 项目

创建maven项目 , groupId = org.mickey , artifactId =hello-spring-boot-starter

```bash
mvn archetype:generate -DgroupId=org.mickey -DartifactId=hello-spring-boot-starter -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DarchetypeCatalog=local	
```

### 引入依赖

1. 继承 spring boot 的 parent
2. 引入 spring-boot-autoconfigure

引入后的pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.mickey</groupId>
    <artifactId>hello-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <relativePath/> 
      	<!-- 版本号 可以自定义-->
        <version>2.2.5.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```



### 编写properties类

```java
@ConfigurationProperties(prefix = "hello")
public class HelloServiceProperties {

    private String msg;

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

### 编写Service

```java
public class HelloService {

    private String msg;

    public String sayHello(){
        return "Hello, " + msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```



### 编写自动配置类

```java
@Configuration // 表示这是一个配置类
@EnableConfigurationProperties(HelloServiceProperties.class) // 开启属性注入
@ConditionalOnClass(HelloService.class) // 当HelloService在classpath下时
@ConditionalOnProperty(prefix = "hello", value = "enabled", matchIfMissing = true) // 当 hello=enabled时properties生效，如果没有默认为true
public class HelloServiceAutoConfiguration {

    @Autowired
    private HelloServiceProperties helloServiceProperties;

    @Bean
    @ConditionalOnMissingBean(HelloService.class) //当容器中没有这个bean时创建Bean
    public HelloService helloService() {
        HelloService helloService = new HelloService();
        helloService.setMsg(helloServiceProperties.getMsg());
        return helloService;
    }
}
```

### 编写 spring.factories

在 src/java/resources/META-INF 下创建 spring.facotries文件

key = org.springframework.boot.autoconfigure.EnableAutoConfiguration

value = HelloServiceAutoConfiguration这个类的全路径

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.mickey.hello.HelloServiceAutoConfiguration
```



这样一个starter就完成了



## 如何使用

在 Spring Boot 项目中引入这个 starter的依赖即可

通过注入 HelloService 直接使用即可 

```java
@Autowired
private HelloService helloService;

@GetMapping("/hello")
public String sayHello() {
  return helloService.sayHello();
}

```

修改配置文件 application.properties

hello.msg = mickey



访问项目

```bash
curl http://localhost:8080/hello 
返回
hello,mickey
```

