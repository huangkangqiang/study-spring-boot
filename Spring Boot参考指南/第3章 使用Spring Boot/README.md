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