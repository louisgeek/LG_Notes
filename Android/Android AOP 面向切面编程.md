# AOP 面向切面编程

- Aspect Oriented Programming
- 部分功能需要横跨各个模块，提高程序的可重用性，集中起来，放到一个统一的地方来控制和管理
- 针对同种问题的统一处理，对业务逻辑的各个部分进行隔离
- 主要用于不想侵入原有代码的场景，耦合度降低
- 提高了开发的效率
- 是 Java Spring 框架中的一个重要内容

则是针对业务处理过程中的**切面进行提取**，它所面对的是处理过程中的某个步骤或阶段，以获得逻辑过程中各部分之间低耦合性的隔离效果。这两种设计思想在目标上有着本质的差异。**AOP并不是与OOP对立的**，而是为了弥补OOP的不足。**OOP解决了竖向的问题，AOP则解决横向的问题**。因为有了AOP我们的调试和监控就变得简单清晰。

简单的来讲，AOP是一种：可以在不改变原来代码的基础上，通过“动态注入”代码，来改变原来执行结果的技术。通过预编译方式和运行期间的动态代理实现

 



# 使用场景

日志打印、统计埋点、性能监控统计、动态权限、安全控制、事物处理、异常处理

日志打印记录、业务数据埋点、性能监测系统、动态权限管理、代码调试



根据注解时机的不同，主要分为运行时、加载时和编译时

源码阶段、class阶段、dex阶段、运行时织入

**源码阶段、class阶段、dex织入**，class加载到虚拟机前，我们统称为静态织入

在运行阶段发生的改动，我们统称为动态织入



在代码的编译期间扫描目标程序，根据切点（PointCut）匹配,将开发者编写的Aspect程序编织（Weave）到目标程序的.class文件中

静态织入

APT，AspectJ、ASM、Javassit

- APT，注解处理器，编译时生成 .java 代码
- 静态织入发生在编译期，因此几乎不会对运行时的效率产生影响

动态织入

 java动态代理，cglib、Javassit

动态织入发生在运行期，可直接将字节码写入内存，并通过反射完成类的加载，所以效率相对较低，但更灵活

java动态代理、cglib只会创建新的代理类而不是对原有类的字节码直接修改，Javassit可修改原有字节码。

其实利用反射或者hook技术同样可以实现代码行为的改变，但由于这类技术并没有真正的改变原有的字节码，所以暂不在谈论范围内，比如xposed，dexposed。



**APT** (Annotation Processing Tool)

后来被annotationProcessor取代





# Java AOP 方案框架

- Eclipse AspectJ
- ASM
- Javassit



## Eclipse AspectJ

- 是一个对 AOP 编程思想的实践
- 作用于 .java 到 .class 层面

### 1 使用 AspectJ 语言

- 一种基于Java平台的面向切面编程的语言，兼容Java平台，可以无缝扩展

- AspectJ语言的语法和Java一样，只是比Java语言多了一些关键字

- ajc（编译器 aspectjtools），基于Java编译器进行扩展，用于编译.aj文件，也可以编译Java代码

- ajc 编译器用来编译aj文件，生成java标准的class文件

### 2 使用 Java 语言、注解

- weaver（织入器 aspectjweaver）

- 通过 Aspect 编织器，编织代码到目标class文件







# Android 使用 AspectJ

- Android Studio 不支持 AspectJ 语言的语法，所以只能采用 Java 注解

  

### 1 Aspect 切面

  

  用 @Aspect 注解一个类，这个类就是针对同种问题的统一处理类，就是一个切面

  

  

  

  

Weaving 编织

## Hook 技术

- 利用 Java 反射实现，动态代理 ，ClassLoader 
- 静态变量或者单例对象，尽量 Hook public 的对象和方法

- 用代理对象替换原始对象，接口可以用动态代理
- Hook 的时机还是尽量要早
- API 版本比较多，方法和类可能不一样，要做好 API 的兼容工作



# 两大类

jni Hook 

静态织入Weave

- APT、AspectJ、ASM、Javassist、DexMaker、ASMDEX
- 在编译的时候织入，对性能无影响
- 指在编译时期就织入，即：编译出来的class文件，字节码就已经被织入了





代码织入

切点表达式，指定了一个时机

分两种

- 需要在切入点上面加注解
- 通过规则切入，不需要加注解







@Aspect

- 声明需要执行的切面类，表示这个类需要 AspectJ 的编译工具  ajc 去编译处理
- AspectJ 配置文件



# 



# 



### 2 Join Point 连接点

- 代码切入点，可以织入代码的地方
- 程序里所有可以织入的地方都是  Join Point 

| Join Point 类型                              | 描述         | 例子 |
| -------------------------------------------- | ------------ | ---- |
| method call                                  | 方法调用     |      |
| method execution                             | 方法执行     |      |
| constructor call                             | 构造方法调用 |      |
| constructor execution                        | 构造方法执行 |      |
| field read access                            | 获取某个变量 |      |
| field write acces                            | 设置某个变量 |      |
| object pre-initialization                    |              |      |
| object initialization                        |              |      |
| class initialization / static initialization |              |      |
| exception handler execution                  | 异常处理     |      |
| advice execution                             |              |      |



@Pointcut

- 声明一个代码切入点，描述哪些注解、方法、类需要进行拦截
- 换句话说就是从 Join Points 中通过某些规则过滤出来的代码切入点





### 3 Pointcut  切入点

- 指定具体的  Join Point 连接点

- 我们关注的点 需要改的点

- 匹配表达式（基于正则表达式），如何插入这些代码

  



```java
// * 通配符
//.. 
// * 表示返回值为任意类型，configTest()代表无参方法
@Pointcut("execution(* configTest())")
// .. 代表方法参数数量任意，类型任意
@Pointcut("execution(* configTest(..))")
//代表方法参数数量一个或多个,最后一个必须是 java.lang.String 类型的，其他参数类型任意
@Pointcut("execution(* configTest(..,java.lang.String))")
//代表方法参数数量一个或多个,第一个必须是 java.lang.String 类型的，其他参数类型任意
@Pointcut("execution(* configTest(java.lang.String,..))")
//代表方法参数有两个,第一个参数类型任意，第二个必须是 java.lang.String 类型的
@Pointcut("execution(* configTest(*,java.lang.String))")
//代表方法参数有两个,第一个参数必须是 java.lang.String 类型的，第二个参数类型任意
@Pointcut("execution(* configTest(java.lang.String,*))")

//匹配 on 开头的方法，比如 onCreate、onResume
@Before("execution(* android.app.Activity.on*(..))")
//
@After("within(@com.test.AspectJxTest *)")
```



#### 自定义 Pointcut

- 自定义注解

  - 通过ProceedingJoinPoint得到注解值
  - 加上@annotation()标志后增加AspectLog参数

  

### 4 Advice 通知

- 编写具体要织入的代码
- Pointcut 选出来的地方进行操作处理
- 拦截后的处理方法



```
[!] [@Annotation] [public,protected,private] [static] [final] 返回值类型 [类名.]方法名(参数类型列表) [throws 异常类型]
```



```
！ 
&&  
|| 
+ 自身以及子类
* 任意类型
.. 任意长度类型
```




```java
@AfterReturning(pointcut = "execution(* com.test.testFun*(..))", returning = "testParam")
public void test(int testParam) {
    Log.e(TAG, "testParam: " + testParam);
}
```



```java
@AfterThrowing(pointcut = "execution(* com.test.testExcep*(..))", throwing = "excep")
public void test(Exception excep) {
    Log.e(TAG, "testParam: " + testParam);
}
```





@Before、@After、@AfterReturning、@AfterThrowing 执行顺序

```java
try {
    //
	try {
	   //@Before 注解修饰的方法 doBefore
		doBefore();
        //反射调用指定方法
		method.invoke();
	} finally {
	  //@After 注解修饰的方法 doAfter
		doAfter();
	}
	//@AfterReturning 注解修饰的方法 doAfterReturning
	doAfterReturning();
    //
} catch (Exception e) {
   //@AfterThrowing 注解修饰的方法 doAfterThrowing
	doAfterThrowing();
}
```

- 声明方法必须是 public 的，返回值必须是 void
- doAfter 在 doAfterReturning 之前执行
- 如果方法抛出异常 doAfterThrowing 会执行



@Around

```java
@Aspect
public class XxxAspect {
	@Around("execution(int xx(..))")
	//参数必须是 ProceedingJoinPoint 类型
	public Object around(ProceedingJoinPoint jp) throws Throwable {
		Object result = null;
		try {
			//
			try {
				//写这的代码相当于 @Before 的功能
				doBefore();
				//可以调用指定方法，也可以不调用，此处类似执行 method.invoke();
				jp.proceed();
			} finally {
			   //写这的代码相当于 @After 的功能
			   doAfter();
			}
			//写这的代码相当于 @AfterReturning 的功能
			doAfterReturning();
			//
		} catch (Throwable e) {
			//写这的代码相当于 @AfterThrowing 的功能
			doAfterThrowing();
		}
		//返回
		return result;
	}
}
```

- 声明方法必须是 public 的
- 所以 @Around 都可以实现 @Before、@After、@AfterReturning、@AfterThrowing 的功能，不能和他们一起使用，否则会重复织入







####  call 和 execution 区别

- call 指的是**调用**，织入在指定方法被调用的位置处
- execution 指的是**执行**，织入到指定方法的内部





Android 中的使用场景

```java
//现成的使用 AOP 的场景
this.registerActivityLifecycleCallbacks
```



Activity 布局加载性能监控

```java
@Around("execution(* android.app.Activity.setContentView(..))")
public void adviceSetContentView(ProceedingJoinPoint joinPoint) throws Throwable {
    String name = joinPoint.getSignature().toShortString();
    long time = System.currentTimeMillis();
    joinPoint.proceed();
    Log.e(TAG, name + " 耗时 " + (System.currentTimeMillis() - time));
}
```

```java
//@Around("execution(* android.app.Activity.setContentView(..))") 打印内容
void androidx.appcompat.app.AppCompatActivity.setContentView(int) 耗时 37
//@Around("execution(* setContentView(..))") 打印内容
void androidx.appcompat.app.AppCompatDelegateImpl.setContentView(int) 耗时 52
void androidx.appcompat.app.AppCompatActivity.setContentView(int) 耗时 52
//@Around("execution(* *..Activity.setContentView(..))") 打印内容
void androidx.appcompat.app.AppCompatActivity.setContentView(int) 耗时 52 
```



target 只能指定类的对象



with 指定目标的所有连接点



### 5 HujiangTechnology AspectJX 使用

```
//采用 AspectJX 来快速配置 Eclipse AspectJ
//project
dependencies {
        classpath "com.android.tools.build:gradle:4.1.2"
        //add
        classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.10'
}
```

```
plugins {
    id 'com.android.application'
    //add
    id 'android-aspectjx'
}
//module
dependencies {
    //add
    implementation 'org.aspectj:aspectjrt:1.9.5'
}
```





