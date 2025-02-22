# Kotlin 基础知识

## Kotlin 特点
- 语法简洁优雅，表现力丰富
- 代码复用性高
- 拥有高级语法特性，支持函数式编程
- 可以编译后运行在不同的环境
- 静态类型编程语言，基于编译器进行代码分析，从而确保类型安全
- 强类型的，支持隐式类型的显式类型语言（类型推断形式实现支持）
- 100% 兼容 Java，支持与 Java 互相操作
- 一切都是对象，没有基础类型，变量也都是引用类型
- Type inference 类型推断
- Null safety 空安全

  
## 为什么 Kotlin 能够开发 Android 应用呢？
首先我们知道 Java 属于解释型语言，Java 代码先被编译成 class 文件，再通过 Java 虚拟机解释成计算机可直接识别的二进制数据后执行，于是这里就引出 Java 虚拟机并不是直接和 Java 代码打交道的，倘若开发一门新语言和新编译器，新编译器能够将这门新语言编译成同种规格的 class 文件，再交由 Java 虚拟机解释运行的话，那这样就可以实现不依赖 Java 语言来开发程序了，而 Kotlin 工作原理就是利用了这种机制，所以 Kotlin 不仅仅能开发 Android 应用，理论上说 Java 能做到的它都能做到



## REPL 交互式编程环境
Read Eval Print Loop 命令行环境





## var、val
variable 可变的变量
value 不可变的变量，写代码时候首选 val

PS: val 对应 java final 修饰的变量，var 对应 java 非 final 修饰的变量



### 类型推断
var d = 1
Int 赋值到 Long 需要显示调用 toLong



### is 进行类型检测判断
- 类似 Java 的 instanceof
- 经过判断后的代码域内其类型就已经进行隐式转换过了，可以直接调用其方法



### as 进行显式类型转换
- 类型不兼容的情况下，使用 as 会抛出转换异常，使用 as? 会得到 null
- 子类不允许转成父类，因为违反了 OOP 



### raw string  原始字符串
三个引号 """  XXX """



### 字符串内模版表达式
$XXX  ${XXX.some}



### 区间
```kotlin
val range = 0..10  //[0,10] 双闭区间
val range = 0 until 10  //[0,10) 左闭右开区间
val range = 10 downTo 0  //降序区间
```



### 流程控制语句
#### if
可以作为一个表达式，支持返回一个值

```kotlin
val c = if (a>b) a else b
```

- 可代替 java 的三目运算符

if 作为代码块，最后一行就是它的返回值

也可以直接在 return 后面跟 if ，省略中间变量

#### when
- 类似 switch ，但更强大
- 和 if 一样，也可以是一个表达式，有一个返回值，也是代码块最后一行的值

- 最后一般都要跟着 else 分支
- 每个分支都可以是一个代码块

```kotlin
var bl="1234"
var bh="test"
when(x){
-1,0 -> doSome
1 -> print("sss")
2 -> {
 	doSomeOther
	}
parseInt(bl) -> print("sasa")
bh -> print("dddd")
in 10..100 -> doSome
!in list -> dododo
else -> print("qqq")
}
```

- 还有一种不带参数的 when



#### for
```
for(po in 0..10){
	print(po)
}
```

```kotlin
for(x in xList){
	print(x)
}
```

遍历带索引

```kotlin
for(i in xList.indices){
	print(xList[i])
}
```

遍历带索引库函数

```kotlin
for((index,value) in xList.withIndex()){
	print(value)
}
```



#### while
和 java 类似



#### break、continue
和 java 类似



#### return
除了表达式可以省略，有返回值的函数都需要显式声明 return



#### label 标签
xxx@ 声明标签

return@xxx 

break@xxx 

this@xxx



#### throw



## 注释
- 可以嵌套



## 标识符

#### 修饰符

#### 关键字

#### this、super

#### 操作符

- 支持重载

- 索引访问操作符  a[i]，list[i]

- 调用操作符a()，a(i) ,就是调用方法！

- 相等、不相等操作符

  - 引用相等 ===，!==，不可重载
  - 结构相等 ==，!=，类似 x.equals(y)

- Elvis 操作符 ?: 是一个二元操作符，跟 null 比较，并且 null 安全

  ```kotlin
  y = x?:0 //类似 y= if( x!== null ) x else 0
  ```

- infix 自定义中缀操作符



### 扩展函数



### 扩展属性



### 空指针安全



### Unit
- 类似 Java 的 void

  

### Any 根类型
- 类似 Java 的 Object



### Nothing
- Nothing? 类似 Java 的 Void



 

### Kotlin data class 没有无参构造函数的问题如何解决？

















### Kotlin 入门 Kotlin primer

Kotlin 的方法可以直接写在文件里

maina 可以快速生成 main 方法

Code -> Convert Java File to Kotlin File 快速转化代码

Tools -> Kotlin -> Show Kotlin Bytecode ，Decompile 按钮查看 kotlin 对应的 java 

Kotlin 没有基本数据类型 Primitive Types，如 int，float 只有 Int(Java 是 Integer)，Float

Kotlin 里面的类都是对象，都继承 `Any `类

Kotlin Int 与 Java int、Java Integer 之间是什么关系？

Kotlin Any 类型与 Java Object 类型之间有什么关系？

## 变量定义 (Defining Variables)

var 变量
val 只读的变量，不是常量，不可变的变量?
以.kt 结尾
代码里的`;`可写可不写
变量名: 变量类型，如 val c: Int
自动推断类型，可以省略 “ : 变量类型 ” ,如 val c =1 

## 空安全 (Null Safety)

String 和 String? 是两种不同的类型， String 可以赋值给 String? 
fun 修饰方法
方法如果返回值为空可以省略
方法没有检查性异常，如果要兼容 java 调用 使用 @Throws 

a?.length 空安全调用，如果 a 为 null ，println(a?.length) 输出 null

Elvis 操作符 ?: ，`var d = b ?: -1` 等价于` var d = if (b != null)  b  else -1`

!! 代表断言，如果确定使用 !! 操作符是有意义的，那尽量靠前使用
不需要 new 关键字



## 类型检查与转换 (Type Checks and Casts)

`if(e is String)` 类似 Java 的 `if(e instanceOf String)`

as 类型转换 as? 空安全的类型转换

## for 循环

支持单行for循环  for (item in list) println(item)

区间 for (i in 1..3)

反区间、步进 for (i in 6 downTo 0 step 2) 

Kotlin val 变量与 Java 的 final 有什么关系？

3. 3 
    UtilsKt 这样的名字如何做到和文件名不一致？
    与java互调
    匿名内部类 object 直接修饰类名的类
    方法默认是 public 的
    java Class<> JavaClazz.class  
    kotlin 里调用 java 的 .class JavaClazz::class.java
    kotlin KClass<>
    repeat 就是对循环的封装
    .. 是 rangeTo 的重载

  如果变量名是关键字，那么需要用 `` 包括起来 

### init 代码块

 init 代码块执行时机在类构造之后，但又在 次构造器 之前

4. 4

   没有封装类，如 Integer

   调用java时候不确定方法返回是否为空，最好用可空类型的变量去接收

   java中只有方法

   kotlin 有函数和方法

   函数嵌套-可以用于递归或者特殊参数 不推荐使用 

   

   ## when

   

   ## 函数 (Functions)

   ```kotlin
   fun fun123(x: Int): Int {
       return 2 * x
   }
   ```

   ## 扩展 (Extensions)

   扩展函数-无法修改的第三方类时候可以使用

   ```kotlin
   fun Context.toast(msg: String, length: Int = Toast.LENGTH_SHORT){
       Toast.makeText(this, msg, length).show()
   }
   //类的实例可空
   fun Context?.toast(msg: String, length: Int = Toast.LENGTH_SHORT){
       if (this == null) { 
           //
       }
       Toast.makeText(this, msg, length).show()
   }
   ```

   

   编译后就是类的静态方法，第一个参数就是扩展函数的实例，而且扩展函数是静态解析的 不具备运行时的多态，因为正在调用的时候是强转的 失去了多态效益

   Kotlin 还支持扩展属性

   ## 继承 (Inheritance)

   Kotlin 的类和方法，默认情况下是 final 的：

   open 修饰类 表示一个类不 final ，可以被继承

   open 修饰方法 表示一个方法可以被重写

   : 表示继承

   override 表示覆写方法

   

   ## 类 (Classes)

   

   ## This 表达式 (Expression)

   inner 修饰内部类，内部类访问外部类方式`this@OutterClass.fun()`

   ###   DSL  

   lambda 是匿名对象

   自带的 支持22个参数的 lambda 

   8 

   inline 常用于修饰高阶函数 使高阶函数实际生成代码变成语句调用 

   9 待复习

   

   ## 数据类 (Data Class)

   ```
   data class Person(val age: Int, val name: String)
   ```

   

   ## 相等性 (Equality)

    java == 相当于 kotlin ===  引用相等，判断是否是同一个对象
   java .equals  相当于 kotlin  ==  或者 .equals  （== 不担心 null 调用）结构相等，判断是否是相同结构

如果一个函数调用的最后一个参数是 lambda 调用时，这时候可以把闭包写在函数括号的外面
在 Java → Kotlin 转换器工作中，始终会把 Java 中的 == 运算符转换成 Kotlin 中的 === 运算符，
出于代码可读性跟功能意图考虑，你需要在必要时您应该把 === 运算符恢复成 == 运算符

suspend 标记当前的函数是可以挂起的

取消前缀来编写目的更为聚焦的函数与类，以便养成更好的编程习惯，
虽然有时候阅读没有前缀的成员变量代码时候会比较费劲，尤其是使用网页版的 Code Review 工具 (比如在很长的类中阅读很长的函数)。不过当您使用 IDE 阅读代码时候，可以通过语法高亮功能很清楚地知道哪些是成员变量，哪些是函数参数



## 委托 (Delegation)

### by lazy 懒加载

### 属性委托

### 函数类型

函数类型 = 参数类型 + 返回值类型

### 高阶函数

**高阶函数就是将函数作为参数或类型返回值类型的函数**





## 函数类型，高阶函数，Lambda表达式三者之间的关系



## 标准函数的原理



```
Cannot access 'androidx.lifecycle.HasDefaultViewModelProviderFactory' which is a supertype of 'com.louisgeek.library.base.BaseActivity'. Check your module classpath for missing or conflicting dependencies
```



```kotlin
//1 === object 
viewModel.organizationList_LD.observe(
            viewLifecycleOwner, object : Observer<List<OrganizationModel>> {
                override fun onChanged(t: List<OrganizationModel>?) {
                
                }
            })

//2 === lambda
 viewModel.organizationList_LD.observe(
            viewLifecycleOwner, Observer {

                
            })
//3 === 
 viewModel.organizationList_LD.observe(
            viewLifecycleOwner,{

                
            })
```

## 可见性修饰符 (Visibility Modifiers)

Java 和 Kotlin 函数可见性修饰符的区别

修饰符 | Java | Kotlin
---|---|---
public | 所有类可见| 所有类可见（默认）
private | 当前类可见 | 当前类可见
protected | 当前类、子类、同一包路径下的类可见 | 当前类、子类可见
default | 同一包路径下的类可见（默认） | 无
internal | 无 | 同一模块中的类可见



```java
public class JavaClazz {  
	public void testMethod1() throws Exception {

	}  
	public void testMethod2() throws Exception {

	}
}
```

```kotlin
class KotlinClazz {
  fun testMethod1() {
      
  }
  fun testMethod2() {
      
  }
}
```



- @Parcelize

  https://bladecoder.medium.com/a-study-of-the-parcelize-feature-from-kotlin-android-extensions-59a5adcd5909

kotlin  1.3.40 及以后 待确认！

```
//启用实验室功能
androidExtensions {
    experimental  = true
}
```


kotlin  1.3.60 及以后

```
//扩展插件 包括 @Parcelize 和 kotlinx.android.synthetic(替代findViewById)
apply plugin: 'kotlin-android-extensions'
```

```
//如果只想启用扩展插件里的 parcelize 功能
androidExtensions {
    features = ["parcelize"]
}
```

kotlin 1.4.20 及以后

```
//独立插件
apply plugin: 'kotlin-parcelize'
```



[Korkel-Client](https://github.com/MarinaB-MB/Korkel)

- hilt

  - @HiltAndroidApp

    

- 泛型 out in

model  和 pojo 写在一个文件里



DATABASE_NAME 写在 AppDataBase 里

- api  -- service

suspend 修饰 fun

- Repository

  ```kotlin
  suspend fun getReposFromApi(
      ): Flow<DataState<List<RepositoryModel>>> = flow {
          try {
              emit(DataState.Loading)
              val reposFromApi = api.getRepos()
              emit(DataState.Success(reposFromApi))
          } catch (e: Exception) {
              e.printStackTrace()
              emit(DataState.Error(e))
          }
      }
  ```



- ViewHolder 结合密闭类

  

- Flow 的 Terminal 运算符

  - collect
  - single/first
  - toList/toSet/toCollection
  - count
  - fold/reduce
  - launchIn/produceIn/broadcastIn

  

  

- 可以理解为 collect() 对应`subscribe()`，而 emit() 对应`onNext()`

- flow 的代码块只有调用 collected() 才开始运行，正如 RxJava 创建的 Observables 只有调用 subscribe() 才开始运行一样。

  

- AppCompatActivity resId 作为参数传入

  

- by viewModels()

  

  

  viewModelScope 解析

  https://blog.csdn.net/wenyingzhi/article/details/97129224

  https://blog.csdn.net/qq_39424143/article/details/105775943

  

- inline safeCall   
- suspend safeCallCoroutine

- ```
  with(viewModel){
  
  }
  ```



#### Gradle Kotlin DSL 替换 Groovy 
[rickgit.github.io](https://github.com/rickgit/rickgit.github.io)

[lzyprime.github.io](https://github.com/lzyprime/lzyprime.github.io)

https://ljd1996.github.io/



