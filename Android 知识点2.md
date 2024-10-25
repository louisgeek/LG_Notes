# Android 知识点


# Java

基础

设计模式大概有哪些，说几个

创建型

抽象工厂 创建者 工厂方法 单例 原型



结构型

适配器 桥接 组合 装饰 外观 代理  享元



行为型

责任链 命令 解释器   备忘录 观察者 访问者 模版方法 迭代器 中介者 状态 策略



 

单例模式：双重校验锁，volatile关键字 

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



# Kotlin



### Kotlin 的协程是什么？

广义上的协程：和线程类似，也是一种解决异步任务的方案

Kotlin 的协程：可以理解是对线程的一种封装，一种可以让异步任务的代码写成一种类似同步的形式，可以看作是一个线程封装框架，官方称为一种轻量级线程

- 可以解决异步任务代码的嵌套
- 必须依赖线程存在，但是可以在不同的线程之间切换
- 由开发者管理，不需要操作系统进行调度和切换



### Kotlin data class 没有无参构造函数的问题如何解决？









# Android

## 消息机制

```java

ActivityThread#main

Looper.prepareMainLooper();

ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

 if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
  }


  Looper.loop();
```



### Handler 是什么？作用是什么？

消息处理器，系统提供一套消息机制实现线程切换，能够让子线程间接的去访问 UI 控件

ActivityThread#main 函数中会调用 Looper.prepare 方法创建一个 Looper , 其构造方法创建了一个 MessageQueue

main 函数继续会执行 Looper.loop 方法，里面利用 while 循环不断从消息队列里取 Message（如果没有消息就利用 Linux epoll 机制进行阻塞等待），然后分发消息

Handler 发消息到消息队列里，而 Message 存着 Handler 的引用，在分发消息的时候会分发到该 Handler 的 dispatchMessage 里处理消息





### 不允许子线程直接去操作 UI 控件是为什么？为什么不采取对 UI 加锁的方式解决？

因为 UI 控件不是线程安全的，子线程去操作可能会导致 UI 的不稳定和不确定性

加锁会让 UI 访问的逻辑变得复杂，而且每次加锁会降低 UI 访问的效率



### IdleHandler

IdleHandler是一个回调接口，可以通过MessageQueue的addIdleHandler添加实现类。当MessageQueue中的任务暂时处理完了（没有新任务或者下一个任务延时在之后），这个时候会回调这个接口，返回false，那么就会移除它，返回true就会在下次message处理完了的时候继续回调。

###  

### 同步屏障

同步屏障可以通过MessageQueue.postSyncBarrier函数来设置。该方法发送了一个没有target的Message到Queue中，在next方法中获取消息时，如果发现没有target的Message，则在一定的时间内跳过同步消息，优先执行异步消息。再换句话说，同步屏障为Handler消息机制增加了一种简单的优先级机制，异步消息的优先级要高于同步消息。在创建Handler时有一个async参数，传true表示此handler发送的时异步消息。ViewRootImpl.scheduleTraversals方法就使用了同步屏障，保证UI绘制优先执行

 

## Activity

### Activity 启动模式

1 standard 标准模式

2 singleTop 栈顶复用模式  （例如：通知栏推送点击消息界面）

3 singleTask 栈内复用模式 （例如：首页）

4 singleInstance 单例模式  （单独位于一个任务栈中，例如：拨打电话界面）

 



### Activity 初始化流程

ActivityThread#startActivityNow
ActivityThread#handleLaunchActivity 处理Activity启动

- ActivityThread#performLaunchActivity 完成Activity启动  -> Activity#onCreate
- Instrumentation#newActivity 实例化Activity
- AppComponentFactory#instantiateActivity App组件工厂实例化Activity
- ClassLoader#loadClass 类加载器
实例化后会调用 Activity#attach 继续进行初始化

ActivityThread#performResumeActivity -> Activity#onResume



### View 绘制入口

ActivityThread#attach

初始化 mWindow

创建顶层布局容器 DecorView 添加到 Window

创建ViewRootImpl 建立 WindowManager和DecorView之间的连接

ViewRootImpl的performTraversals

performMeasure、performLayout、performDraw

依次执行 measure、layout、draw 三大流程

measure用来对View进行测量

layout 负责将子元素在父元素中的位置即真实宽高以及四个顶点位置

draw 负责将View绘制出来



### View 初始化

Activity#setContentView

- PhoneWindow#setContentView
- PhoneWindow#installDecor 
- PhoneWindow#generateDecor 初始化 DecorView



### View 绘制流程

View的绘制从ActivityThread类中Handler的处理RESUME_ACTIVITY事件开始，在执行performResumeActivity之后，创建Window以及DecorView并调用WindowManager的addView方法添加到屏幕上，addView又调用ViewRootImpl的setView方法，最终执行performTraversals方法，依次执行performMeasure，performLayout，performDraw。也就是view绘制的三大过程。
 measure过程测量view的视图大小，最终需要调用setMeasuredDimension方法设置测量的结果，如果是ViewGroup需要调用measureChildren或者measureChild方法进而计算自己的大小。
 layout过程是摆放view的过程，View不需要实现，通常由ViewGroup实现，在实现onLayout时可以通过getMeasuredWidth等方法获取measure过程测量的结果进行摆放。
 draw过程先是绘制背景，其次调用onDraw()方法绘制view的内容，再然后调用dispatchDraw()调用子view的draw方法，最后绘制滚动条。ViewGroup默认不会执行onDraw方法，如果复写了onDraw(Canvas)方法，需要调用 setWillNotDraw(false);清楚不需要绘制的标记。



### ViewRootImpl

建立 DecorView 和 Window 之间的联系

invaliate



### 触发 View（DecorView） 第一次绘制的流程

ActivityThread#handleResumeActivity
ActivityThread#performResumeActivity
获取 ViewManager（getWindowManager 转化来的，ViewManager是一个接口，这里其实是 WindowManagerImpl ，就是上文【PhoneWindow 初始化】里创建的）
ViewManager#addView 添加 decorView（getDecorView 来的），

- WindowManagerImpl#addView 添加 DecorView 
直接调用 WindowManagerGlobal#addView，内部初始化 ViewRootImpl 
ViewRootImpl#setView 设置 decorView
ViewRootImpl#requestLayout
- DecorView.assignParent(ViewRootImpl);
-ViewRootImpl 是一个 ViewParent（不是 view，不要和 parent view概念混淆？）
ViewRootImpl是WindowManager和DecorView之间的桥梁，measure测量，layout布局，draw绘制三大流程都是它完成的



### requestLayout 流程

上文提到 requestLayout 那么说下它的流程
View#requestLayout
ViewParent#requestLayout
DecorView 是整个 View 层级的最顶层，ViewRootImpl 又是 DecorView 的 parent，所以最终调用到 ViewRootImpl 的 requestLayout
ViewRootImpl#requestLayout
ViewRootImpl#scheduleTraversals
Choreographer#postCallback 传入 mTraversalRunnable
Runnable 执行 ViewRootImpl#doTraversal
ViewRootImpl#performTraversals
ViewRootImpl#performMeasure

- View#measure
ViewRootImpl#performLayout
- View#layout
- 执行调用过 requestLayout 的 View 的 measure 和 layout
- 将还没有执行的 requestLayout 加到队列中，下一次 frame 中进行执行
ViewRootImpl#performDraw
- ViewRootImpl#draw



### Measure

#### MeasureSpec

int 32 位  2 位 SpecMode + 30 位 SpecSize 

SpecMode 

- UnSpecified 
- Exactly
- at_most



DecorView 的 MeasureSpec 由 Window 的大小和自己的 LayoutParams 决定

View 的 MeasureSpec 由父容器的大小和自己的 LayoutParams 决定

 View -> onMeasure -> setdis -> setdisraw

ViewGroup -> onMeasure(子 View ) -> setdis -> setdisraw



#### Layout

View 用 View#layout 确定自身位置

View   View#layout

ViewGroup 需要调 onLayout 确定子 View 的位置

ViewGroup   View#layout ->  View#onLayout 

### Draw

画背景

onDraw 进行绘制自己

dispatchDraw 绘制子 View





### Activity 启动流程

startActivity最终都会调用startActivityForResult，通过ActivityManagerProxy调用system_server进程中ActivityManagerService的startActvity方法，如果需要启动的Activity所在进程未启动，则调用Zygote孵化应用进程，进程创建后会调用应用的ActivityThread的main方法，main方法调用attach方法将应用进程绑定到ActivityManagerService（保存应用的ApplicationThread的代理对象）并开启loop循环接收消息。ActivityManagerService通过ApplicationThread的代理发送Message通知启动Activity，ActivityThread内部Handler处理handleLaunchActivity，依次调用performLaunchActivity，handleResumeActivity（即activity的onCreate，onStart，onResume）







 

### 获取 LayoutInflater 实例的方式

- LayoutInflater 布局解析器，用于解析布局文件后动态生成 View

- 是一个抽象类

- 是一个系统服务



1 Context#getSystemService

2 LayoutInflater#from

3 Activity#getLayoutInflater



三种获取实例的方法最终都是通过 Context#getSystemService 方法实现的，里面涉及到单例模式

Activity#setContentView 方法内部也用到了 LayoutInflater 进行布局解析

LayoutInflater 内部采用了 org.xmlpull 解析器实现 xml 解析



### LayoutInflater#inflate 的流程

- 有一个 View#inflate 方法，就是调用 LayoutInflater#inflate 两个参数的方法
- 内部会自动 addView 所以不适合用在 Adapter 和加载 Fragment 布局等情况

1 LayoutInflater 有四个重载方法，其中两个针对 xml 布局，另外两个是针对现成的 XmlResourceParser

2 如果是 xml 布局文件，可以通过 Resources#getLayout 把 xml 布局文件解析加载得到 XmlResourceParser

3 因为 XmlResourceParser 包含属性信息，所以利用 Xml#asAttributeSet 将其转化成 AttributeSet 供后续使用

4 通过 advanceToRootNode 方法找到根标签，如遇到的不是 <merge/> 标签就通过 createViewFromTag 方法把根标签对应类通过反射的方式创建出来，然后调用 rInflateChildren 方法，而它就是调用了 rInflate 方法，从而实现了递归，而 rInflate 内部也调用了 createViewFromTag ，所以用递归调用的方式把子 View 一个个创建出来并通过 addView 加入其父 View 中，如遇到 <merge/> 标签就直接调用 rInflate 先进行处理一次，最终形成一个视图树

5 整个流程中还涉及到反射、ClassLoader、换肤等知识点可以展开



### Activity#setContentView 原理

## fixme

```
ActivityThread#performLaunchActivity ->
Activity#attach ->  mWindow = new PhoneWindow(this, window, activityConfigCallback)



```


#### 层级关系

```java
Activity -> mPhoneWindow -> mDectorView(继承自FrameLayout) -> 
mContentRoot(LinearLayout) -> mContentParent(FrameLayout) ->  Custom Layout View
```



#### 流程

1 每个 Activity 都包含一个 Window ，而 Window 是抽象类，PhoneWindow 是他的唯一继承实现类，Activity#attach 里初始化 PhoneWindow

2 Activity#setContentView 直接调用 PhoneWindow#setContentView 方法

3 PhoneWindow 又包含了一个 DecorView ，是继承自 FrameLayout  ，作为界面的根布局，在 PhoneWindow#installDecor 中的 PhoneWindow#generateDecor 进行初始化

4 通过主题样式确定加载哪个系统的 xml 文件作为 DecorView 的子布局 mContentRoot，默认是 screen_simple.xml ，根节点是 LinearLayout，在 DecorView#onResourcesLoaded 里进行 xml 解析

5 screen_simple.xml 包括一个 actionBar 标题栏 ViewStub，和一个带系统 id 的 @android:id/content 的 FrameLayout，代码对应 mContentParent，作为加载布局的容器，在 PhoneWindow#generateLayout 中初始化

6 然后内部也是用 LayoutInflater#inflate 通过 LoadXmlResourceParser 把 xml 布局解析成 View

## fixme

绘制则交给了 ViewRootImpl 来完成，通过 performTraversals() 触发绘制流程，performMeasure 方法获取View 的尺寸，performLayout 方法获取 View 的位置 ，然后通过 performDraw 方法遍历View 进行绘制



#### PhoneWindow

1 每个 Activity 都包含一个 Window ，而 PhoneWindow 是 Window 的唯一继承实现类

2 Activity 的展示界面的特性是通过 Window 对象来处理的

3 提供了一系列绘制窗口的方法，如设置背景、标题

4 是 Activity 和 View 交互的桥梁



### PhoneWindow 初始化

上文 【Activity 初始化流程】提到了
ActivityThread#performLaunchActivity 实例化Activity后会调用
Activity#attach 

- 实例化 PhoneWindow
- ps:并且通过 Window#setWindowManager 给 Window 设置了一个WindowManager（WindowManager是一个接口，其实是 WindowManagerImpl）



#### DecorView

1 作为整个应用窗口(Activity界面)的顶层布局容器，可以说是所有 View 的 parent view 

2 是 Android 最基本的窗口系统

3 继承自 FrameLayout，就是对 FrameLayout 进行功能的修饰

4 将要显示的具体内容呈现在 PhoneWindow 上



### Activity、Window 、View 之间关系

一个 Activity 对应一个 Window，也就是 PhoneWindow ，在 PhoneWindow 中有一个 DecorView，在setContentView 中会将 layout 填充到这个 DecorView 中





### 自定义 View









## 事件分发

MotionEvent 事件产生后，按照 Activity ->  Window -> DectorView -> View 顺序传递的过程就叫时间分发

#### dispatchTouchEvent

分发事件



#### onInterceptTouchEvent (只有 ViewGroup 有)

拦截事件



#### onTouchEvent

处理事件



OnTouchListener -> OnTouchEvent -> OnClick





### Android 操作系统启动流程





### App 启动流程





### Android 资源加载机制



### 



### AsyncTask 异步任务

- 内部通过 Handler 实现



### Binder





## ListView 



## RecyclerView 

### RecyclerView 和 ListView 的区别

- RecyclerView 原生支持纵向、横向和瀑布流等布局，ListView 只支持纵向布局
- RecyclerView 利用 ItemDecoration 实现分割线，较灵活，ListView 利用 android:divider
- RecyclerView 自带 ViewHolder 机制，ListView 不强制要求实现 ViewHolder 机制
- RecyclerView 自带 Item 动画效果，ListView  没有
- RecyclerView 自带局部刷新
- RecyclerView 自带拖拽、侧滑接口
- RecyclerView 未实现头尾视图、空视图和 Item 点击事件
- RecyclerView 改进了缓存机制，有四级缓存
- RecyclerView  可扩展性强



### RecyclerView 缓存机制

四级缓存：

mChangedScrap

mAttachedScrap、mHiddenViews、mCachedViews

ViewCacheExtension 自定义缓存

RecycledViewPool 需要重新绑定数据





### DiffUtil

- 实现 DiffUtil#Callback 定义新旧数据以及 Item 数据的比较规则
- 通过 DiffUtil#calculateDiff 计算得到 DiffResult （推荐放在子线程）
- 利用 DiffResult#dispatchUpdatesTo 通知刷新

注意：1 需要解决 Adapter 与 DiffUtil 的数据要一致性问题，每次刷新需要同时设置 Adapter 和 DiffUtil 的数据

​			2 定义维护新旧数据的时候需要注意引用传递问题



### 已经有 Adapter#notifyItemXXX 局部刷新了为啥还要有 DiffUtil 工具类？

- DiffUtil 最终也是调用 Adapter#notifyItemXXX 局部刷新方法进行刷新，只是工具类帮你计算了，不需要自己去计算维护哪些 position 需要刷新

- 如果只改变了个别 Item ，那么直接调用 Adapter#notifyItemXXX 即可，那就没必要用 DiffUtil 了

- DiffUtil 设计不单单是只能和 RecyclerView 一起使用，只要实现 ListUpdateCallback 接口的都可以用它来计算新旧数据集差异




### AsyncListDiff

- 通过 XxxAdapter 封装 AsyncListDiffer 进行使用，内部实现在子线程中执行 DiffUtil#calculateDiff  方法
- 实现 DiffUtil#ItemCallback 定义 Item 数据的比较规则，内部自行维护了新旧数据集
- 利用 AsyncListDiff#submitList 进行数据更新，同时自动完成刷新



### 已经有 DiffUtil  了为啥还要 AsyncListDiff ？

- 无需自行维护新旧数据集，解决了 Adapter 与 DiffUtil 的数据引用的一致性问题和数据引用传递问题
- 内部已经实现了通过子线程进行差异计算，解决了主线程阻塞问题
- 调用数据更新的时候会自动刷新接口，避免遗忘





### androidx.recyclerview.widget.ListAdapter

- 可以让 XxxAdapter 直接继承 ListAdapter ，简化了 AsyncListDiff 【通过 XxxAdapter 封装 AsyncListDiffer 进行使用】这一步骤

按顺序选型：

Adapter#notifyDataSetChanged -> 尽量选用局部刷新 Adapter#notifyItemXXX -> 数据集变化较复杂就用 DiffUtil 配合处理 -> 数据集较大可以选用 AsyncListDiffer 处理 -> 不想封装 Adapter 就直接继承 ListAdapter 处理





### ViewHolder

主要解决了 ItemView 复用的问题，减少重复 findViewById 的消耗



## 缓存机制

### 三级缓存

内存、硬盘、网络







### LRU 缓存

LRU ：最近最少使用



## OKHttp



## Retrofit 

### 请求接口有多个 BaseUrl 的话要怎么处理？

-  最基础的就是用不同的 baseUrl 新建不同的 Retrofit 实例





## Jetpack

### Lifecycle 





### App Startup

很多三方库都用 ContentProvider 的小技巧进行 library 的初始化，如 LeakCanary

那如果大家都用这种方式的话，那就要初始化很多 ContentProvider ，一定程度上会影响性能，官方就出了这个，为了就是让所以类似需求统一到一个 ContentProvider 里去初始化

 



## 框架

### 使用 mvp 时遇到的坑



## Material



ShapeableImageView

- 自带，可实现圆角等效果



























### 安卓各版本大变化

Android 5.0

Material Design
ART虚拟机
Android 6.0

应用权限管理
官方指纹支持
Doze电量管理
运行时权限机制->需要动态申请权限
Android 7.0

多窗口模式
支持Java 8语言平台
需要使用FileProvider访问照片
安装apk需要兼容
Android 8.0

通知,渠道->适配
画中画
自动填充
后台限制
自适应桌面图标->适配
隐式广播限制
开启后台Service限制
Android 9.0

利用 Wi-Fi RTT 进行室内定位
刘海屏 API 支持
多摄像头支持和摄像头更新
不允许调用hide api
限制明文流量的网络请求 http
Android 10

暗黑模式
隐私增强(后台能否访问定位)
限制程序访问剪贴板
应用黑盒
权限细分需兼容
后台定位单独权限需兼容
设备唯一标示符需兼容
后台打开Activity 需兼容
非 SDK 接口限制 需兼容







# 计算机网络



## 浏览器中输入一个 URL  然后回车访问，这个过程中间发生了什么？

- DNS 解析，把网址解析找到域名对应的 IP 地址
- 客户端发起 HTTP 请求，会进行 TCP 三次握手建立连接，服务端发送报文给客户端
- 客户端收到报文解析得到 HTML页面等数据，然后构建 DOM 树，再构造呈现树并绘制到浏览器页面上面



TCP/IP 协议



TCP 三次握手，四次挥手，为什么要这样？



UDP 协议，和 TCP 的区别



前后台传输数据需要用密钥对数据加密，那加密过程应该放在哪个位置



​	