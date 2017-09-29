# 使用Spring Boot

## 构建系统

建议选择一个支持依赖管理，能消费发布到“Maven中央仓库”的artifacts的构建系统，比如Maven或Gradle。使用其他构建系统也是可以的，比如Ant，但它们可能得不到很好的支持。

### 依赖管理

spring boot每次发布时都会提供一个它所支持的精选依赖列表。实际上，在构建配置中不要提供任何依赖的版本，因为spring boot已经管理好了。当更新spring boot时，那些依赖也会一起更新。

> 如果有必要，可以指定依赖的版本来覆盖spring boot默认版本。

精选列表包括所有能够跟spring boot一起使用的spring模块及第三方库，该列表可以在spring-boot-dependencies中获取到。

>spring boot每次发布都会关联一个spring框架的基础版本，所以建议不要自己指定spring版本。

### Maven

maven用户可以继承spring-boot-starter-parent项目来获取合适的默认设置。该parent项目提供一下特性：

+ 默认编译级别为java 1.6
+ 源码编码为UTF-8
+ 一个dependency management节点，允许省略常见依赖的<version>标签，继承自spring-boot-dependencies pom
+ 恰到好处的资源过滤
+ 恰到好处的插件配置（exec插件，surefire，git commit id，shade）
+ 恰到好处的对application.properties和application.yml进行筛选，包括特定的profile（profile-specific）的文件，比如application-foo.properties和application-foo.yml。

最后一点：由于配置文件默认接收spring锋哥的占位符`${...}`，所以maven filtering需改用`@...@`占位符（你可以使用maven属性resource.delimiter来覆盖它）。

#### 继承starter parent

配置项目，让其继承自spring-boot-starter-parent，配置如下：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.1.BUILD-SNAPSHOT</version>
</parent>
```

> 只需在该依赖上指定spring boot版本，如果导入其他的starters，可以省略版本号。

按照以上配置，可以在项目中通过覆盖属性来覆盖个别的依赖。例如，可以在pom中设置，将spring data升级到另一个版本。

```xml
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

#### 改变java版本

spring-boot-starter-parent选择了相当保守的java兼容策略，如果使用最新的java版本，可以添加一个java.version属性：

```xml
<properties>
    <java.version>1.8</version>
</properties>
```

#### 使用spring boot maven插件

spring boot包含一个maven插件，它可以将项目打包成一个可执行jar，可以将该插件添加到<plugin>节点：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin<artifactId>
        </plugin>
    <plugins>
</build>
```

注意：如果使用spring boot parent pom，只需添加该插件而无需配置它，除非需要改变定义在parent中的设置。

## 组织代码

### 使用"default"包

当类没有生命package时，它被认为处于default package下。通常不推荐使用default package，因为对于使用@ComponentScan，@EntityScan或@SpringBootApplication注解的spring boot应用来说，它会扫描每个jar中的类，这会造成一定问题。

> 建议遵循java推荐的包命名规范，使用一个翻转的域名（com.example.project）。

### 放置应用的main类

通常建议将应用的main类放到其他类所在包的顶层，并将@EnableAutoConfiguration注解到main类，这样就隐式地定义了一个基础的包搜索路径，以搜索某些特定的注解实体（比如@Service，@Controller等）。例如，一个jpa应用，Spring将搜索@EnableAutoConfiguration注解的类所在包下的@Entity实体。

采用root package的方式，就可以使用@ComponentScan注解而不需要指定basePackage属性，也可以使用@SpringBootApplication注解，只要将main类放到root package中。

Application.java将声明main方法，还有基本的@Configuration。

```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

## 配置类

spring boot提倡基于java的配置。尽管可以使用xml调用SpringApplication.run()，不过还是建议你使用@Configuration类作为主要配置源。通常定义了main方法的类也是使用@Configuration注解的一个很好的替补。

> 虽然网上有很多使用xml配置的spring示例，建议尽可能的使用基于java的配置，搜索查看`enable*`注解就是一个很好的开端。

### 导入其他配置类

不需要将所有的@Configuration放进一个单独的类，@Import注解可以用来导入其他配置类。另外，可以使用@ComponentScan注解自动搜集所有spring组件，包括@Configuration类。

### 导入xml配置

如果必须使用xml配置，建议从一个@Configuration类开始，然后使用@ImportResource注解加载xml配置文件。

## 自动配置

spring boot自动配置尝试根据添加的jar依赖自动配置到spring应用。例如，如果classpath下存在HSQLDB，并且没有手动配置任何数据库连接的beans，那么spring boot将自动配置一个内存型数据库。

实现自动配置有两种可选方式，分别是将@EnableAutoConfiguration或@SpringBootApplication注解到@Configuration类上。

> 建议只添加一个@EnableAutoConfiguration注解，通常建议将它添加到主配置类上。

### 逐步替换自动配置

自动配置是非侵入性的，任何时候都可以定义自己的配置类来替换自动配置的特定部分。例如，添加自己的DataSource bean，默认的内嵌数据库支持将不被考虑。

如果需要查看当前应用启动了哪些配置项，可以在运行应用时打开 --debug 开关，这将为核心日志开启dubug日志级别，并将自动配置相关的日志输出到控制台。

### 禁用特定的自动配置项

如果发现启用了不想要的自动配置项，可以使用@EnableAutoConfiguration注解的exclude属性禁用它们：

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

如果该类不在classpath中，可以使用该注解的excludeName属性，并指定全限定名来达到相同效果。最后，可以通过spring.autoconfigure.exclude属性exclude多个自动配置项。

> 通过注解级别或exclude属性都可以定义排除项。

## spring beans和依赖注入

可以自由地使用任何标准的spring框架技术去定义beans和它们注入的依赖。简单起见，经常使用@ComponentScan注解搜索beans，并结合@Autowired构造器注入。

如果遵循以上的建议组织代码结构（将应用的main类放到包的最上层，即root package），那么就可以添加@ComponentScan注解而不需要任何参数，所有应用组件（@Component，@Service，@Repository，@Controller等）都会自动注册成spring beans。

下面是一个@Service bean的示例，它使用构造器注入获取一个需要的RiskAssessor bean。

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...
}
```

> 注意使用构造器注入允许riskAssessor字段被标记为final，这意味着riskAssessor后续是不能改变的。

## 使用@SpringBootApplication注解

经常使用@Configuration，@EnableAutoConfiguration，@ComponentScan注解的main类，Spring Boot提供了一个方便的@SpringBootApplication注解代替。

@SpringBootAppliction注解等价于默认属性使用@Configuration，@EnableAutoConfiguration和@ComponentScan

```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

> @SpringBootApplication注解提供了用于自定义@EnableAutoConfiguration和@ComponentScan属性的别名。

运行应用程序

将应用打包成jar，并使用内嵌http服务器的一个最大好处是，你可以像其他方式那样运行你的应用程序。调试Spring Boot应用也很简单，都不需要任何特殊IDE插件或扩展。

> 本章节只覆盖基于jar的打包，如果选择将应用打包成war文件，最好参考相关的服务器和IDE文档。