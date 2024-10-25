# Java Inversion of Control 控制反转
- 简称 IOC，是一种设计思想（设计原则）
- 实现控制反转常见的两种方式：Dependency Injection 依赖注入和 Service Locator Pattern 服务定位器模式
- Java 中实现 IOC 主要综合使用了工厂模式、Xml 解析、反射等技术
- 控制：指控制对象的创建和销毁，即控制对象的生命周期
- 反转：指依赖对象的获得方式被反转了，以前是客户端直接 new 出依赖项的实例，现在则是转移交给了 IOC 容器去创建和控制

## Dependency Injection 依赖注入
- 简称 DI，又叫依赖项注入
- 是实现 IOC 思想的一种方法，通过将依赖关系从代码内部移动到代码外部，让外部实体（如 IOC 容器）负责创建和管理对象及其依赖关系
- 依赖注入常见的两种方式：Constructor Injection 构造器（构造函数）注入和 Field Injection 属性（字段）注入（Setter Injection 注入）
- 依赖项注入以控制反转原则为基础根据该原则，通用代码控制着特定代码的执行，复用代码，易于重构，易于测试
 
### 手动依赖注入
1. 最原始的在使用的地方按顺序实例化依赖项然后注入，意味着业务逻辑处出现了大量样板代码
2. 自行创建一个依赖项容器类，比如叫 AppContainer，由于各个地方都有可能用到它，所以比如可以写在 Application 类，这意味着必须自行管理 AppContainer，手动为所有依赖项创建实例，仍存在部分样板代码

### 自动依赖注入
1. 基于 Java 反射的解决方案，可以在运行时连接依赖项，比如 Guice，可能存在性能问题
2. 静态解决方案，可在编译时生成连接依赖项的代码，比如 Google 的 Dagger 或者 Hilt（Dagger 和 Hilt 可以共存）