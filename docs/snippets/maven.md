### 创建Java项目

```shell
mvn archetype:generate -DgroupId=org.dark -DartifactId=NumberGenerator -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false -DarchetypeCatalog=local
```



### 创建Java Web项目

```shell
mvn archetype:generate -DgroupId=com.yiibai.web -DartifactId=ProjectName -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false -DarchetypeCatalog=local
```



### Maven compiler plugin

```xml
 <plugins>
   <plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-compiler-plugin</artifactId>
     <version>3.7.0</version>
     <configuration>
       <source>1.8</source>
       <target>1.8</target>
     </configuration>
   </plugin>
</plugins>
```



### 将本地jar安装到本地maven仓库

```shell
mvn install:install-file -Dfile=jar包位置 -DgroupId=com.c
mbc.socketservice -DartifactId=com.cmbc.socketservice -Dversion=1.0.0.RELEASE -D
packaging=jar
```




