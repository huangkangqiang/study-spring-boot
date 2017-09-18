# 使用Spring Boot

## 构建系统

建议选择一个支持依赖管理，能消费发布到“Maven中央仓库”的artifacts的构建系统，比如Maven或Gradle。使用其他构建系统也是可以的，比如Ant，但它们可能得不到很好的支持。

### 依赖管理

spring boot每次发布时都会提供一个它所支持的精选依赖列表。实际上，在构建配置中不要提供任何依赖的版本，因为spring boot已经管理好了。当更新spring boot时，那些依赖也会一起更新。

> 如果有必要，可以指定依赖的版本来覆盖spring boot默认版本。

精选列表包括所有能够跟spring boot一起使用的spring模块及第三方库，该列表可以在spring-boot-dependencies中获取到。

>spring boot每次发布都会关联一个spring框架的基础版本，所以建议不要自己指定spring版本。
