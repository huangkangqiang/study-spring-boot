# 开始

## Spring Boot介绍

Spring Boot简化了基于Spring的应用开发，只需要“run”就能创建一个独立的、产品级别的Spring应用。我们为Spring平台及第三方库提供开箱即用的设置，这样你就可以有条不紊地开始。多数Spring Boot应用只需要很少的Spring配置。

可以使用Spriong Boot创建java应用，并使用 java -jar 启动它或者采用传统的war部署方式。我们也提供了一个运行Spring脚本的命令行工具。

主要目标：

+ 为所有的Spring开发提供一个从根本上更快，且随处可得的入门体验。
+ 开箱即用，但通过不采用默认配置可以快速摆脱这种方式。
+ 提供一系列大型项目常用的非功能性特征，比如：内嵌服务器，安全，指标，健康检测，外部化配置。
+ 绝对没有代码生成，也不需要XML配置。

## 系统要求

建议使用Java8，尽管可以通过配置在Java6或Java7环境下可以使用。

## Spring Boot安装

Spring Boot可以跟经典的Java开发工具一起使用或安装成一个命令行工具。

查看当前安装的Java版本：

> $ java -version

## 支持依赖管理的构建工具

### Maven安装

### Gradle安装

##　Spring Boot CLI安装

Spring Boot CLI是一个命令行工具，可用于快速搭建基于Spring的原型。它支持Groovy脚本，这也意味着可以使用类似Java的语法，但不用写很多的模板代码。

Spring Boot不一定非要配合CLI使用，但它绝对是Spring应用取得进展的最快方式。

### 手动安装

Spring CLI分发包可以从Spring软件仓库下载。

下载完成后，解压，根据INSTALL.txt操作指南进行安装。

配置环境变量，使用一下命令可以查看CLI版本：

> $ spring --version

## Spring CLI实例快速入门

通过以下一个相当简单的web应用，可以测试Spring CLI安装是否成功。创建一个名叫app.groovy的文件：

```groovy
@RestController
class ThisWillActullyRun{

    @RequestMapping("/")
    String home(){
        "Hello World!"
    }
    
}
```

然后运行以下命令：

> $ spring run app.groovy

首次运行该应用将会花费一些时间，因为需要下载依赖，后续运行将会快很多。

使用浏览器打开[localhost:8080](http://localhost:8080)，然后就能看到输出：

> Hello World!

## 开发你的第一个Spring Boot应用

项目采用Maven进行构建，因为大多数ide都支持。

项目开始之前，先查看jdk和maven版本是否可用：

> $ java -version

> $ mvn -v

### 创建pom

```xml
<?xml version="1.0" encoding="UTF-8">
<project 
    xmlns="http://maven.apache.org/POM/4.0.0" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.BUILD-SNAPSHOT</version>
    </parent>
    <!-- Additional lines to be added here... -->
    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

目前为止，一个可工作的构建就完成了。可以通过`mvn package`测试。

可以将项目导入到ide中。

### 添加classpath依赖

Spring Boot提供很多starters，用来简化添加jars到classpath的操作。示例程序中已经在pom的parent节点使用了`spring-boot-starter-parent`，它是一个特殊的starter，提供了有用的Maven默认配置。同时，它也提供了一个`dependency-management`节点，这样对于期望的依赖就可以省略`version`标记了。

查看下目前的依赖：

> $ mvn dependency:tree

其他starters只简单提供开发特定类型应用所需的依赖。由于正在开发web应用，我们将添加`spring-boot-starter-web`依赖。

`mvn dependency:tree`命令可以将项目依赖以树形方式展现出来，可以看到`spring-boot-starter-parent`本身并没有提供依赖。添加`spring-boot-starter-web`依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

再次运行`mvn dependency:tree`，可以看到多了一些其他依赖，包括Tomcat web服务器和Spring Boot自身。

### 编写代码

为了完成应用程序，需要创建一个单独的java文件。meven默认会编译src/main/java下的源码，所以需要创建相应的文件结构，并添加一个名为src/main/java/Example.java的文件：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```

####　@RestController和@RequestMapping

@RestController，这被称为构造型（stereotype）注解。它为阅读代码的人提供暗示（这是一个支持REST的控制器），对于Spring，该类扮演了一个特殊角色。Example类是一个web @Controller，所以当web请求进来时，Spring会考略是否使用它进行处理。

@RequestMapping注解提供路由信息，它告诉Spring任何来自"/"路径的HTTP请求都应该被映射到`home`方法。@RestController注解告诉Spring以字符串的形式渲染结果，并直接返回给调用者。

#### @EnableAutoConfiguration

第二个类级别的注解是@EnableAutoConfiguration，这个注解告诉spring boot根据添加的jar依赖猜测如何配置spring。由于`spring-boot-starter-web`添加了tomcat和spring mvc，所以`auto-configuration`将假定正在开发一个web应用，并对spring进行相应的设置。

starters和Auto-Configuration：Auto-Configuration设计成可以跟starters一起很好的使用，但这两个概念没有直接的联系。你可以自由地挑选starters以外的jar依赖，spring boot仍会尽最大努力去自动配置你的应用。


