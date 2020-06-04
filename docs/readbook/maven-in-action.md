# Maven 实战

## maven是什么

一个优秀的构建工具

## maven 能干什么

帮助开发者构建项目

### maven的优秀在哪？

提供一个完整的构建生命周期模型 （意味着构建的标准化）

maven提供了成熟的插件机制





### 核心概念 (core concepts)

#### maven生命周期

maven 的生命周期分为3个阶段

clean   清理

default 编译打包，安装部署 都是这个阶段的一部分

site 生成站点



#### maven坐标

通过 `groupId` `artifactId` `version` 这三个元素确定一个唯一的坐标



#### 依赖管理

`groupId` `artifactId` `version` 这三个元素可以确定唯一的坐标，可以指定这三个元素的值去引入对应的依赖

比如要引入MySql的依赖只需要这样

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.version}</version>
</dependency>
```



#### 依赖传递

A依赖B，B依赖C， A也会间接的依赖C

简单理解就是 A引入了B的依赖，会同时将C依赖进来





#### 聚合

#### import

#### 插件