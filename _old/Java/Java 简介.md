# Java 是 Java 平台和 Java 语言的总称

- Java 平台
  - 跨平台

- Java 语言
  -  Sun Microsystems 太阳微系统公司(升阳公司)在 1995 年 5 月 23 日发布的高级编程语言



# 发行版本

- Java Standard Edition      Java SE  

- Java Enterprise Edition     Java EE 

- Java Micro Edition     Java ME

  (2005 年 6 月在 JavaSE 6 发布时开始改名，之前分别叫 J2SE、J2EE 、J2ME )





# JDK

- Java Development Kit Java开发工具包



# JRE

- Java Runtime Environment  Java 运行时环境



# JVM

- Java Virtual Machine  Java 虚拟机



# 环境变量

- JAVA_HOME 定义 JDK 所在的路径

```
D:\DevTools\Java\jdk1.8.0_202
```

- CLASSPATH 指定**类搜索路径**，声明 Java 程序执行时从哪里去寻找所需要的类（可能涉及到 ClassLoader ？）

```
//从 JDK 5 开始就默认会到当前目录及当前目录 lib 目录下去寻找，所以不用设置 CLASSPATH 也能正常编译和运行 Java 程序 （也有文章说是 JDK1.3 以后？）
// . 代表当前目录，.. 代表上一级目录(父目录)
.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar
```

- PATH 指定**命令搜索路径**，声明命令提示符执行命令时可以在哪些路径里去查找对应程序

```
  %JAVA_HOME%\bin;
  //JRE 环境变量（可选） 
  %JAVA_HOME%\jre\bin;
```

  

# 事件记录

1995 年 5 月 23 日发布

2004 年 9 月 30 日发布 Java 1.5 时直接命名 Java 5

2009 年 4 月 20 日甲骨文收购了 Sun 公司

2011 年 7 月 28 日 Java 7 发布



# Java 特性

- 简单的

继承了 C 和 C++ 的优点，去掉了难点，复杂点：类多继承，操作符重载，自动强制类型转换，加入了引用代替指针，自动垃圾回收内存管理

- 面向对象的

类 接口 继承

类单继承

接口多继承

类实现接口

动态绑定

- 健壮的

强类型机制

异常处理

丢弃指针

垃圾自动回收

安全检查机制

-  跨平台可移植的

java 编译器用 java 写的

java 运行环境用 c 实现

java 程序编译成 字节码 ，java 解释器进行解释运行



- 分布式

- 安全的
- 高性能
- 多线程
- 支持类动态加载
- 运行时类型检查