## Java

Java 属于编译型语言还是解释性语言

Java 语言的运行机制是 Java 代码编译后生成一种特殊的 class 文件，这个格式的文件只有 Java 虚拟机才能识别，所以这个 Java 虚拟机就充当了解释器的角色，将 class 文件解释成计算机可直接识别的二进制数据后进行执行，所以 Java 属于解释性语言，Python 和 JavaScript 也属于解释性语言，C/C++ 属于编译型语言





设计模式大概有哪些，说几个

创建型

抽象工厂 创建者 工厂方法 单例 原型



结构型

适配器 桥接 组合 装饰 外观 代理  享元



行为型

责任链 命令 解释器   备忘录 观察者 访问者 模版方法 迭代器 中介者 状态 策略



 

单例模式：双重校验锁，volatile 关键字 

- ### volatile 关键字

  1. 保证可见性,不保证原子性

  2. 禁止指令重排序

  3. 不缓存,每次都是从主存中取

     https://www.cnblogs.com/zhengbin/p/5654805.html

  ### hashmap 原理



### 红黑树



二叉树



### jvm内存



- Java堆





### 类加载器

Android平台上虚拟机运行的是Dex字节码,一种对class文件优化的产物,传统Class文件是一个Java源码文件会生成一个.class文件，而Android是把所有Class文件进行合并，优化，然后生成一个最终的class.dex,目的是把不同class文件重复的东西只需保留一份,如果我们的Android应用不进行分dex处理,最后一个应用的apk只会有一个dex文件。
 Android中常用的有两种类加载器，DexClassLoader和PathClassLoader，它们都继承于BaseDexClassLoader。区别在于调用父类构造器时，DexClassLoader多传了一个optimizedDirectory参数，这个目录必须是内部存储路径，用来缓存系统创建的Dex文件。而PathClassLoader该参数为null，只能加载内部存储目录的Dex文件。所以我们可以用DexClassLoader去加载外部的apk。

 



### String,StringBuffer,StringBuilder 区别





### 线程池

- 对线程统一管理，统一调度的工具，可以复用已存在的线程，减少线程的创建和销毁的开销，降低消耗，增加速度

- 能够控制并发的线程数的上限，既能够降低资源的消耗浪费，也一定程度上避免了系统处理时候的堵塞，提高了系统资源的合理利用率，增强了对线程的管理性

- 能够实现定时周期任务

  ​	

### 内置线程池

FixedThreadPool

SingleThreadPool

ScheduleThreadPool

CachedThreadPool

### 自定义线程池





注解是什么，元注解又是什么？

注解：是针对代码的一种标注、描述、解释，可以针对代码来配置一些功能，能够用来自动生成 Java 代码



元注解：修饰注解的注解就叫元注解

- @Target
- @Retention
- @Inherited
- @Documented



### effectively final

- 事实上地、实际上地

java 8 后，匿名内部类、Lambda 表达式访问的局部变量不需要再显示声明成 final 了

[为什么必须是final的呢？ (cuipengfei.me)](https://cuipengfei.me/blog/2013/06/22/why-does-it-have-to-be-final/)





### effective java

- 有价值的，有效的



- google

  - guava

    https://github.com/google/guava

    Google Guava官方教程（中文版）http://ifeve.com/google-guava/

- java 集合

JAVA集合类汇总http://www.cnblogs.com/leeplogs/p/5891861.html

- Float Double

JAVA float double数据类型保留2位小数点5种方法
http://www.cnblogs.com/simpledev/p/4765834.html

Float或Double浮点型计算导致小数点后面显示过长数据的解决方法

 http://blog.csdn.net/bbg_0622/article/details/5616094

java float、double精度研究（转）
http://www.cnblogs.com/chenfei0801/p/3672177.html

Java浮点数float，bigdecimal和double精确计算的精度误差问题总结
http://www.cnblogs.com/wangyt223/p/6210916.html



- 零基础学Java 04

java 类名和文件名必须一致

- 零基础学Java 05

  cmd --> dir 命令

  

- 零基础学Java 10

int 区间 -2^31 ~ 2^31-1  



- 零基础学Java 11

表达式 语句 代码块

整数的字面值默认是 int

关键字

标识符

运算符

字面值

数据类型

变量

表达式 

语句  ;

代码块  {  }



- 零基础学Java 12

bit  和 byte

二进制的位 叫 bit

byte 是计算机存储单元

整数类型

byte -128 ~ 127

short (2 byte) -32768 ~ 32767

int (4 byte)

long (8 byte)

浮点（小数）类型

float (4 byte)

double (8 byte)

布尔类型

1 byte

char 类型 --- 单引号

2 byte



- 零基础学Java 13



算术运算符

比较运算符

布尔运算符



！ 非 not

& 且 and

&& 且且 and and 

| 或 or

|| 或 或 or or 













