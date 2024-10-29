# Android 知识点

# 四大组件
典型的 Android 应用包含多个应用组件 app component ，包括 Activity、Fragment、Service、内容提供程序 content provider 和广播接收器 broadcast receiver，应用组件可以不按顺序地单独启动，并且操作系统或用户可以随时销毁它们，因此不应在应用组件中存储任何应用数据或状态，并且应用组件不应相互依赖


## 已经有 Activity 了，为什么还要引入 Fragment
Fragment 允许将界面分成为好几个区块，从而将模块化和可重用性引入 
Activity 
同一功能界面根据屏幕大小不同可以实现两个版本的页面显示样式

## Handler 消息机制的原理

子线程中创建 Handler

Handler post 方法原理

HandlerThread

## Linux 进程间通信
管道

消息队列

共享内存

套接字

信号量
## 跨进程通信方式


## !!!!Binder IPC 通信大概分了几层
一、Java 层：供应用程序开发者使用的 Java 接口层，通过这一层可以方便地进行进程间通信。
二、Native 层 JavaBBinder 和 BpBinder：Native 层中，，用于实现 Java 层与内核层之间的通信转换。
三、Native 层 Binder 驱动：。
四、服务端和客户端通信层：服务端实现具体的服务逻辑并通过 Binder 机制提供给客户端调用，客户端通过 Binder 代理与服务端进行通信。

Framework 框架层：这一层主要包含Java Binder和JNI（Java Native Interface）部分，用于上层应用程序与Binder框架的交互。
Native 层：创建 Service Manager 以及 BBinder、BpBinder 模型，负责实际的 IPC 通信过程，Java 层的 Binder 对象对应 Native 层的 JavaBBinder 对象，Java 层的 BinderProxy 对象对应 Native 层的 BpBinder 对象
Kernal 层：对应就是 Binder 驱动，位于 Linux 内核中，负责处理进程间通信的底层实现，管理 Binder 实体对象和引用，以及处理进程间的数据传输和同步等操作


## 哪些场景会涉及到 Binder IPC 通信
- 1 当应用程序需要访问系统服务（如 ActivityManager、PackageManager、WindowManager 和 ContentProvider等）时，会通过 Binder 机制与系统服务进行通信
    - ActivityManager.getService 获取 ActivityManagerService 实例
    - Context.getContentResolver 获取 ContentResolver 实例
- 2 startActivity、startService、bindService 和 registerReceiver 等方法会涉及与系统的 ActivityManagerService 等服务进行 Binder 通信
- 3 通过 ContentResolver.query 等方法查询数据时，会与对应的 ContentProvider 进行 Binder 通信以获取数据
- 4 生命周期回调，比如 onCreate , onStart , onResume 等
- 5 插件化框架中的通信应用
- 6 AIDL 就是通过 binder 实现跨进程通信




##  !!!!!!JobScheduler


##  !!!!!!后台任务解决方案
- 如果是一个长时间的 http 下载的话就使用 DownloadManager
- 否则的话就看是不是一个可以延迟的任务，如果不可以延迟就直接使用 Foreground service
- 如果可以延迟的话就看是不是可以由系统条件触发，如果是的话就使用 WorkManager
- 如果不是就看是不是需要在一个固定的时间执行这个任务，如果是的话就使用 AlarmManager
- 如果不是的话就还是使用 WorkManager
- 




Gradle的实现，gradle中task的生命周期


WebSocket与socket的区别？

vm的运行时数据结构。栈帧中会有什么异常？方法区里面存放的是什么数据

插件化原理
Android 插件化原理

利用 ContentProvider 实现初始化 library 获取 Context




## Google Volley
- https://github.com/google/volley
- https://developer.android.google.cn/training/volley

## Google Guava
- https://github.com/google/guava

## Square Picasso
- https://github.com/square/picasso
- https://square.github.io/picasso


## Square Okio2
- https://github.com/square/okio
- https://square.github.io/okio




# EventBus 3.1.1 源码解析
https://dev-xu.cn/posts/1a3301f8.html




# Android 知识点







# Android

- 



## Android 消息机制

- Android 的消息机制在 Java 层及 Native 层均是由 Handler、Looper、MessageQueue 三者构成

Android 消息机制是 Android 系统提供一套消息机制，主要是用于实现线程切换，能够让子线程间接的去访问 UI 控件



### Handler 是什么？

Handler 消息处理器，既是消息发送者也是消息处理者，Handler 发消息到 MessageQueue 消息队列里，而消息队列中的 Message 消息中存放着的 Handler 引用，在分发消息的时候会将消息分发到该 Handler 的dispatchMessage 里进行处理



### Looper 是什么?

- ActivityThread#main 方法中通过 Looper#prepareMainLooper 调用 Looper#prepare 创建一个 Looper , 涉及 ThreadLocal 存取 Looper ，而 Looper  的构造方法里创建了一个 MessageQueue

- ActivityThread#main 方法继续执行，里面有个 Looper.loop 方法，内部利用 while 循环在主线程中开启了一个死循环，用它不断得从 MessageQueue 消息队列里取 Message（如果没有消息就利用 Linux epoll 机制进行阻塞等待），然后分发消息给 target Handler 处理

- 每个线程只能有一个 Looper

```java
static volatile Handler sMainThreadHandler;
//ActivityThread#main
public static void main(String[] args) {
	Looper.prepareMainLooper();

	ActivityThread thread = new ActivityThread();
	thread.attach(false, startSeq);

	if (sMainThreadHandler == null) {
 	  sMainThreadHandler = thread.getHandler();
	}


	Looper.loop();
    //主线程的死循环一旦退出，就会抛异常，本意就是在正常情况下得让该方法一直处于死循环状态
	throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

```java
 public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                //只能创建一次，主线程有且仅有一个 Loooper
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
```


### Handler 是什么？作用是什么？

消息处理器，系统提供一套消息机制实现线程切换，能够让子线程间接的去访问 UI 控件


























ActivityThread#main 函数中会调用 Looper.prepare 方法创建一个 Looper , 其构造方法创建了一个 MessageQueue

main 函数继续会执行 Looper.loop 方法，里面利用 while 循环不断从消息队列里取 Message（如果没有消息就利用 Linux epoll 机制进行阻塞等待），然后分发消息

Handler 发消息到消息队列里，而 Message 存着 Handler 的引用，在分发消息的时候会分发到该 Handler 的 dispatchMessage 里处理消息




### MessageQueue 是什么？

用于存储 Message，内部维护了 Message 的链表，每次拿取 Message 时，若该 Message 离真正执行还需要一段时间，会通过 nativePollOnce 进入阻塞状态，避免资源的浪费。若存在消息屏障，则会忽略同步消息优先拿取异步消息，从而实现异步消息的优先消费



### ThreadLocal 是什么?

- ThreadLocal 是一个线程内部的数据存储类，在指定的线程中存取数据，别的线程无法获取该线程的数据，实现线程的数据隔离

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有指定线程中可以获取到存储的数据，对于其它线程来说无法获取到数据。一般开发情景中我们用不到ThreadLocal，但是在一些非常复杂或者特殊的情景中， 通过ThreadLocal可以轻松地实现一些看起来很复杂的功能。

在Android的系统机制中，以下机制中都使用到了ThreadLocal：Looper、ActivityThread、ActivityManagerService。

**一般来说，当我们管理数据的时候是以线程为作用域并且不同线程管理不同的数据副本的时候，就可以考虑使用ThreadLocal。ThreadLocal的作用就是提供了一个全局的哈希表，用于实现指定线程的数据控制。**

具体到ThreadLocal的使用场景，一般可以概况为，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal。



### 主线程的消息队列是否有上限?





### 为什么不允许子线程直接去操作 UI 控件？不采取对 UI 加锁的方式解决？

因为 UI 控件不是线程安全的，子线程去操作可能会导致 UI 的不稳定和不确定性

加锁会让 UI 访问的逻辑变得复杂，而且每次加锁会降低 UI 访问的效率



### IdleHandler

IdleHandler是一个回调接口，可以通过MessageQueue的addIdleHandler添加实现类。当MessageQueue中的任务暂时处理完了（没有新任务或者下一个任务延时在之后），这个时候会回调这个接口，返回false，那么就会移除它，返回true就会在下次message处理完了的时候继续回调。

###  

### 同步屏障

同步屏障可以通过MessageQueue.postSyncBarrier函数来设置。该方法发送了一个没有target的Message到Queue中，在next方法中获取消息时，如果发现没有target的Message，则在一定的时间内跳过同步消息，优先执行异步消息。再换句话说，同步屏障为Handler消息机制增加了一种简单的优先级机制，异步消息的优先级要高于同步消息

- 在创建 Handler 时，Handler 构造函数有一个 async 参数，默认传 false，若 async 传 true 则表示这个 Handler 发出的消息均为异步消息，消息会被优先处理

ViewRootImpl.scheduleTraversals方法就使用了同步屏障，保证UI绘制优先执行



### 可以让自己发送的消息优先被执行吗？原理是什么？

这个问题，我感觉只能说：在有同步屏障的情况下是可以的。

同步屏障作用：在含有同步屏障的消息队列，会及时的屏蔽消息队列中所有同步消息的分发，放行异步消息的分发。

在含有同步屏障的情况，我可以将自己的消息设置为异步消息，可以起到优先被执行的效果。



### Looper 主线程存在死循环，为什么没有卡死？没有发生ANR？

先说下ANR：5秒内无法响应屏幕触摸事件或键盘输入事件；广播的onReceive()函数时10秒没有处理完成；前台服务20秒内，后台服务在200秒内没有执行完毕；ContentProvider的publish在10s内没进行完。所以大致上Loop死循环和ANR联系不大，问了个正确的废话，所以触发事件后，耗时操作还是要放在子线程处理，handler将数据通讯到主线程，进行相关处理。

线程实质上是一段可运行的代码片，运行完之后，线程就会自动销毁。当然，我们肯定不希望主线程被over，所以整一个死循环让线程保活。

为什么没被卡死：在事件分发里面分析了，在获取消息的next()方法中，如果没有消息，会触发nativePollOnce方法进入线程休眠状态，释放CPU资源，MessageQueue中有个原生方法nativeWake方法，可以解除nativePollOnce的休眠状态，ok，咱们在这俩个方法的基础上来给出答案

- 当消息队列中消息为空时，触发MessageQueue中的nativePollOnce方法，线程休眠，释放CPU资源
- 消息插入消息队列，会触发nativeWake唤醒方法，解除主线程的休眠状态

- 当插入消息到消息队列中，为消息队列头结点的时候，会触发唤醒方法
- 当插入消息到消息队列中，在头结点之后，链中位置的时候，不会触发唤醒方法

- 综上：消息队列为空，会阻塞主线程，释放资源；消息队列为空，插入消息时候，会触发唤醒机制

- 这套逻辑能保证主线程最大程度利用CPU资源，且能及时休眠自身，不会造成资源浪费

- 本质上，主线程的运行，整体上都是以事件（Message）为驱动的





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

  内部会自动 addView 所以不适合用在 Adapter 和加载 Fragment 布局等情况

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



### Window

- 是一个抽象类，唯一继承实现类是 PhoneWindow

- 描述一个窗口，是一种抽象概念，不是真实存在的，Window 实例也是以 View 的形式存在的

- Window 包含 View，对 View 进行管理



### WindowManager

- 是一个接口类，继承 ViewManager 接口，实现类是 WindowManagerImpl

- 对 Window 进行管理，包括增加、更新、删除操作（定义在 ViewManager 里）

- WindowManagerImpl 方法内部又是调用 WindowManagerGlobal 的方法的，涉及到桥接模式的知识



### WMS 作用

- WindowManager 具体的操作都通过 WMS 处理实现的，所以 WMS 主要功能是 View 的最终管理者
- 和 WindowManager 之间是通过 Binder 实现跨进程通信的
- WMS 负责窗口管理，关联 WindowManager
- WMS 负责窗口动画管理，关联 WindowAnimator
- 输入系统的中转站，关联 InputManagerService
- Surface 管理，关联 SurfaceFlinger



### Window、WindowManager、WMS 之间的关系

Window 包含并管理 View，WindowManager 管理 Window，WindowManager 的操作通过 WMS 实现的



### Window 的类型

Application Window 应用程序窗口

- 如 Activity

Sub Window 子窗口

- 不能独立存在，需要依附在其他窗口存在，如 PopupWindow

System Window 系统窗口

- 如 Toast、系统音量条窗口



### PhoneWindow

1 每个 Activity 都包含一个 Window ，而 PhoneWindow 是 Window 的唯一继承实现类

2 Activity 的展示界面的特性是通过 Window 对象来处理的

3 提供了一系列绘制窗口的方法，如设置背景、标题

4 是 Activity 和 View 交互的桥梁



### PhoneWindow 初始化

上文 【Activity 初始化流程】提到了
ActivityThread#performLaunchActivity 实例化 Activity 后会调用
Activity#attach 

- 实例化 PhoneWindow
- ps:并且通过 Window#setWindowManager 给 Window 设置了一个 WindowManager（WindowManager 是一个接口，其实是 WindowManagerImpl）



#### DecorView

1 作为整个应用窗口(Activity界面)的顶层布局容器，可以说是所有 View 的 parent view 

2 是 Android 最基本的窗口系统

3 继承自 FrameLayout，就是对 FrameLayout 进行功能的修饰

4 将要显示的具体内容呈现在 PhoneWindow 上



### Activity、Window 、View 之间的关系

一个 Activity 对应一个 Window，也就是 PhoneWindow ，在 PhoneWindow 中有一个 DecorView，在setContentView 中会将 layout 填充到这个 DecorView 中



### Context#getSystemService

 其实就是 ContextImpl#getSystemService 内部通过从 ArrayMap 中获取

ArrayMap 里的值是在静态代码块中通过 SystemServiceRegistry#registerService 存入的

```java
static {
...
    registerService(Context.ACTIVITY_SERVICE, ActivityManager.class,
                new CachedServiceFetcher<ActivityManager>() {
            @Override
            public ActivityManager createService(ContextImpl ctx) {
                return new ActivityManager(ctx.getOuterContext(), ctx.mMainThread.getHandler());
            }});
...
registerService(Context.WINDOW_SERVICE, WindowManager.class,
                new CachedServiceFetcher<WindowManager>() {
            @Override
            public WindowManager createService(ContextImpl ctx) {
                //这里是 Impl 实现类，一个参数
                return new WindowManagerImpl(ctx);
            }});
...
}
```



### 











## 自定义 View









## 事件分发

MotionEvent 事件产生后，按照 Activity ->  Window -> DectorView -> View 顺序传递的过程就叫事件分发



#### dispatchTouchEvent

分发事件

ViewGroup#dispatchTouchEvent

- 判断是否需要拦截事件

#### onInterceptTouchEvent (只有 ViewGroup 有)

拦截事件



#### onTouchEvent

处理事件



OnTouchListener -> OnTouchEvent -> OnClick







## 启动流程

### Android 系统启动流程

```
接通电源，按下启动按钮
ROM 中引导芯片代码开始执行
加载 BootLoader 到 RAM 执行
BootLoader 负责拉起系统 OS
Linux 内核启动，完成系统设置，查找 init.rc 文件，启动 init 进程
初始化和启动属性服务，启动 Zygote 进程
创建 Java 虚拟机并注册 JNI 方法，创建名为"zygote"的 Server Socket，启动 SystemServer 进程
创建 Binder 线程池和 SystemServiceManager,启动各种系统服务
启动 Launcher ，显示已安装 APP 列表
```



#### init 进程启动流程

init.rc 配置文件中， AIL 语句中 Import 语句引入  init.${ro.zygote}.rc 文件，如 init.zygote64.rc

```
init.cpp#main
- mount 挂载启动所需的文件目录
- property_init
- signal_handler_init 子进程信号处理函数，会收到进程终止的消息，防止 init 进程的子进程成为僵尸进程
- start_property_service
- parser 解析 init.rc 配置文件
-- AIL 语句中 Service 语句完成 service.cpp#ServiceManager#AddService 加入配置的服务到 vector  列表
-- AIL 语句中 Action 语句完成 builtins.cpp#ServiceManager 进行 ForEach 遍历得到 Service,然后执行 service.cpp#Start
--- 利用 service.cpp#folk 函数创建 Service 子进程，service.cpp#execve 函数启动该子进程,并会进入到 Service 的 main 函数里，ps：如果 service 是 Zygote 类型，就相当于进入了 Zygote 进程的 main 函数里
```

init 进程

- Android 系统用户空间的第一个进程

- 创建和挂载启动所需的文件目录

- 初始化和启动属性服务

- 解析 init.rc 配置文件，并启动 Zygote 进程（ init.zygote64.rc 系列配置文件）



##### 属性服务

类似 Windows 注册表管理器，以键值对的形式存储系统和软件的信息

```
init.cpp#main 中的 start_property_service
- property_service.cpp#create_socket 创建非阻塞的 Socket 返回 property_set_fd 
- property_service.cpp#listen 函数传入 property_set_fd，形成 server 型 Socket
- property_service.cpp#register_epllo_handler 利用 linux 的 epoll 来监听 property_set_fd ，并通过 property_service.cpp#handle_property_set_fd 函数来处理，内部又调用了 property_service.cpp#handle_property_set 函数进行处理
-- 通过是否有 "ctl."前缀来分成两种类型，一种控制 handle_control_message ，一种普通 property_set 
```



##### Zygote 进程启动流程

```
接上文【init 进程启动流程】最后已经进入到 Service 的 main 函数里了
如果 init 进程解析的是 init.zygote64.rc 文件，那么就是进入了 Zygote 进程的 main 函数里了
app_main.cpp#main
- AndroidRuntime.cpp#start "com.android.internal.os.ZygoteInit"
-- startVm 启动 Java 虚拟机
-- startReg 注册 JNI 方法
-- toSlashClassName 替换 . 为 /
-- FindClass 找到 ZygoteInit.java 类
-- GetStaticMethodID 得到 ZygoteInit#main 方法
-- CallStaticVoidMethod 通过 JNI 方式调用该 main 方法
接着进入到 ZygoteInit#main 方法里
- ZygoteServer#registerServerSocket 创建名为"zygote"的 Server Socket
- ZygoteInit#preload 预加载类和资源
- ZygoteInit#startSystemServer，对应类名 "com.android.server.SystemServer"
- ZygoteServer#runSelectLoop 等待 AMS 请求创建新的应用程序进程
```



Zygote 进程

- Android.mk 定义名字 "app_process "，进程启动后，Linux 系统下的 pctrl 系统调用 app_process ，将名字改成 "zygote"
- init 进程启动时创建了 Zygote 进程
- Zygote 进程启动时会创建 ART（Dalvik）
- 通过 AndroidRuntime.cpp#start 函数启动 Zygote 进程(ZygoteInit)
- 创建 Java 虚拟机，并为 Java 虚拟机注册 JNI 方法
- 通过 JNI 调用 ZygoteInit#main 方法，ZygoteInit#main 是 Java 框架层面的入口
- 利用 registerServerSocket 创建 Server Socket ，runSelectLoop 等待 AMS 请求，预加载类和资源
- 并按需利用 fork 自身创建应用程序进程和 SystemServer 进程
- ps：ZygoteInit#main 间接调用了 ActivityThread#main



###### SystemServer 进程启动流程

```
ZygoteInit#main
- ZygoteInit#startSystemServer
-- ZygoteInit#handleSystemServerProcess
--- ZygoteInit#createPathClassLoader
--- ZygoteInit#zygoteInit
---- ZygoteInit#nativeZygoteInit 通过调用 c 层代码在新进程中创建 Binder 线程池
----- AndroidRuntime.cpp 里函数继续调用 app_main.cpp#onZygoteInit 函数，然后是 app_main.cpp#startThreadPool 
---- RuntimeInit#applicationInit
----- RuntimeInit#invokeStaicMain 通过反射得到 SystemServer 类，对应类名 com.android.server.SystemServer，以及他的 main 方法，最后通过抛异常的方式间接调用了 SystemServer#main 方法
- new SystemServer().run()
-- Looper#prepareMainLooper 是不是和之前学的基础 ActivityThread#main 里的套路相识？
-- System.loadLibrary("android_services") 加载 so 库
-- createSystemContext
-- new SystemServiceManager
-- SystemServer#startBootstrapServices 引导服务，启动了 AMS、PowerManagerService
-- SystemServer#startCoreServices 核心服务，启动了 BatteryService
-- SystemServer#startOtherServices 其他服务，启动了 WMS、CameraService、PackageManagerService
三类服务启动方式类似，并且会在服务 List 的集合中 add 进去
SystemServiceManager#startService
-SystemService#onStart
ps：另外也可以通过各个服务自身的 main 方法实例化自己
```

SystemServer 进程

- 启动新进程的 Binder 线程池
- 创建系统服务管理类 SystemServiceManager

- 创建各种系统服务，如 AMS、WMS、PackageManagerService



###### -> Launcher 桌面启动器启动流程

Launcher 的入口是 AMS 的 systemReady 方法

```
接上文【SystemServer 进程启动流程】，SystemServer#startOtherServices 启动其他服务
内部会调用 ActivityManagerService#systemReady
接着一系列调用后到达 ActivityManagerService#startHomeActivityLocked
然后是 ActivityStarter#startHomeActivityLocked
最终是 Launcher#onCreate 方法
- LauncherModel#loadWorkspace
- LauncherModel#bindWorkspace
- LauncherModel#loadAllApps 加载所有已安装 APP 的信息
AllAppsContainerView#onFinisnInflate 
- 找到 id 为 apps_list_view 的 RecyclerView 把 apps 列表的数据显示出来
```



###### 应用程序进程启动流程

上文【Zygote 进程启动流程】Zygote 进程的 Server Socket 等待 AMS 请求创建新的应用程序进程

应用程序启动的条件是应用程序进程是否已经启动，而 AMS 在启动应用程序的时候会先检查这个应用程序需要的应用程序进程是否已经启动，如未启动则会请求 Zygote 进程去先启动这个应用程序进程

```
========================= AMS 发送请求到 Zygote =================================
AMS 利用 ActivityManagerService#startProcessLocked 方法通过建立 Socket 连接发送请求给 Zygote 进程，提供一个app 的uid 和 entryPoint="android.app.ActivityThread"，他就是应用程序进程主线程的类名
- Process#start 尝试启动进程
-- 调用 ZygoteProcess#start
--- ZygoteProcess#startViaZygote
========================= Zygote 处理 AMS 请求 ==================================
Zygote 进程在 ZygoteServer#runSelectLoop 方法下等待 AMS 的请求
然后利用 Zygote#forkAndSpecialize 方法 fork 方式创建应用程序进程
接着调用 ZygoteConnection#handeChildProc
- ZygoteInit#zygoteInit 接下来就和上文【SystemServer 进程启动流程】中段流程基本一致了
-- ZygoteInit#nativeZygoteInit 通过调用 c 层代码在新进程中创建 Binder 线程池
--- AndroidRuntime.cpp 里函数继续调用 app_main.cpp#onZygoteInit 函数，然后是 app_main.cpp#startThreadPool 
-- RuntimeInit#applicationInit
--- RuntimeInit#invokeStaicMain 通过反射得到 ActivityThread 类，对应类名 android.app.ActivityThread，以及他的 main 方法，最后通过抛异常的方式间接调用了 ActivityThread#main 方法
- Looper#prepareMainLooper 上文【SystemServer 进程启动流程】中提到的问号
- thread = new ActivityThread 主线程管理类
- sMainThreadHandler = thread.getHandler 用于处理主线程的消息循环
- Looper#loop
```

应用程序进程

- 启动新进程的 Binder 线程池
- 创建主线程管理类 ActivityThread
- 创建一个用于处理主线程的消息循环



### App 启动流程(根 Activity 启动流程)

```
在 Launcher 中点击 App 图标启动应用
- Launcher#startActivitySafely 
-- Activity#startActivity 带上 FLAG_ACTIVITY_NEW_TASK 标记，基础知识
--- Activity#startActivityForResult
---- Instrumentation#execStartActivity
----- ActivityManager.getService().startActivity 利用 AIDL 进程间通信方式与 AMS 通信，最终就是调用到 ActivityManagerService#startActivity
然后后面经过一系列调用会到 ActivityStarter#startActivityLocked 是不是和 Launcher 中间一段类似？
- ActivityStarter#startActivity 
- ActivityManagerService#getRecordForAppLocked 得到 Launcher 的 ProcessRecord
- new ActivityRecord
- 这个 ProcessRecord 就是需要传入的要启动 Activity 所在的应用程序进程
走到后面如果这个 ProcessRecord 为 null，表示应用程序进程未启动，那么走 ActivityManagerService#startProcessLocked 通知 Zygote 去启动该应用程序进程，上文【应用程序进程启动流程】提到的知识得到印证了
ProcessRecord 不为空的话经过一系列调用会到 ActivityThread$ApplicationThread.thread.scheduleLaunchActivity
小结一下：
1 Launcher 点击， 通过 AIDL ，Binder 通信方式到 AMS
2 而 AMS 是在 SystemServer 进程中的
3 AMS 想调用 ActivityThread$ApplicationThread 的方法
4 如果 ActivityThread$ApplicationThread 所在进程（应用程序进程）未启动，AMS 通过 socket 方式通知 Zygote 进程去完成对应进程的创建
5 如果已经创建了，则 AMS 通过 Binder 方式和 ActivityThread$ApplicationThread 所在进程进行通信，请求启动根 Activity
回到 scheduleLaunchActivity 继续
ActivityThread#sendMessage 通过 Handler 调用 H#sendMessage 方式发送通知启动对应 Activity
通过 Handler 的回调 H#handleMessage 方法处理调用 ActivityThread#handleLaunchActiviy 接下来的逻辑就比较基础了，是不是很熟悉？
- ActivityThread#performLaunchActivity 内部创建 Activity 的 Context ,利用类加载器通过 Instrumentation#newActivity 创建对应 Activity 的实例,也创建了 Application
-- Activity#attach 内部创建了 PhoneWindow 
-- Instrumentation#callActiviyOnCreate
--- Activity#preformCreate
---- Activity#onCreate
- ActivityThread#handleResumeActivity
```

启动流程涉及到 4 个进程：

Launcher 进程点图标请求 AMS 所在进程 SystemServer 进程请求启动应用程序根 Activity

AMS 判断该应用程序的应用程序进程是否已经启动

如果未启动则 AMS 通知请求 Zygote 进程先去创建并启动这个应用程序进程

应用程序进程启动后的逻辑就是 AMS 通知应用程序进程里的 ActivityThread$ApplicationThread 启动根 Activity





### AMS 的作用

负责四大组件的启动、切换、调度

应该程序进程的管理、调度



### Instrumentation 的作用







### Android 资源加载机制








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





## 性能优化

### 绘制优化

FPS Frames Per Second

肉眼在 60fps 以上就不会感觉卡顿

1/60=16.667



Profile GPU Rendering

通过彩色柱状图来表示出渲染问题，可在设备开发者选项里打开



显示 GPU 过渡绘制

通过彩色线条来表示多度绘制，可在设备开发者选项里打开



Systrace

数据采样分析工具，可在 DDMS 中打开使用，也可以在代码中编写 Trace 指定语句使用



TraceView

SDK 中自带的数据采集分析工具，可在 DDMS 中打开使用，也可以在代码中编写 Debug 指定语句使用



Hierarchy Viewer

SDK 自带可视化布局调试工具



Android Lint

代码扫描工具



<include/>、<merge/>、<ViewSub/>

include 对引用优化，可以把通用布局抽象出来

merge 合并布局，一般和 include 一起使用

ViewSub 通过 android:layout 指定待加载布局文件，java code 利用 inflate 触发加载，不过只能加载一次，不能实现显示与隐藏切换，不能嵌套<merge/>，ViewSub 操作的是布局



### 内存优化

#### 内存泄漏

内存不足时候，Android 运行时就会触发 GC ，GC 采用的垃圾标记算法是根搜索算法，GC Roots 根对象集合

内存泄漏即：没有用的对象到 GC Roots 仍旧是可达的，意味着被引用，导致 GC 不能回收该无用的对象

1 非静态内部类的静态实例

非静态内部类会持有外部类的引用，如果该非静态内部类存在有静态实例的话，那么该静态实例生命周期比 Activity 长，那它持有的 Activity 引用导致 Activity 无法回收

2 多线程相关的匿名内部类/非静态内部类

匿名内部类也会持有外部类的引用，异步任务如果耗时，Activity 如果已经销毁了，那它持有 Activity 的引用就导致 Activity 无法及时回收

3 Handler 内存泄漏

Handler 的 MessageQueue 里的部分 Message 如果长时间存在，则会导致 Handler 无法及时回收

非静态匿名内部类的 Handler 会隐性引用外部 Activity ,导致 Activity 等无法及时回收

-  改用静态内部类
- 改成弱引用
- 在 Activity 销毁的时候手动将 MessageQueue 清空

4 未正确使用 Context

如果将Activity 实例传入，单例类的方法，容易造成内存泄漏，因为单例类生命周期比它长

5 静态 View

静态 View 往往持有 Activity 的引用，在 Activity 销毁的时候记得置空

6 WebView

Android 系统自身的缺陷，最好是单独开一个进程操作 WebView

7 资源对象未关闭

File 等一般都使用缓冲，需要在用完后及时关闭，置空

8 集合内对象未清理

及时将集合中无用的引用移除，特别是静态集合

9 Bitmap 对象

较大的 Bitmap 用完及时释放，尽量避免静态变量持有较大的 Bitmap

10 监听器未移除

注册、反注册注意按需成对，Listener 注意移除

11 ThreadLocal 内存泄漏

ThreadLocalMap中的Entry中，ThreadLocal作为key，是作为弱引用进行存储的。当ThreadLocal不再被作为强引用持有时，会被GC回收，这时ThreadLocalMap对应的ThreadLocal就变成了null。而根据文档所叙述的，当key == null时，这时就可以默认该键不再被引用，该Entry就可以被直接清除，该清除行为会在Entry本身的set()/get()/remove()中被调用。

一般情况下，线程在执行结束时，自然也会消除其对value的引用，使得Value能够被GC回收。但是在使用线程池或者其他特殊情况下的时候，会存在线程复用的情况，这时候Value的值依然能获取到，可能就存在内存泄漏的隐患了。所以我们推荐通过手动将value的值设置为null（即调用ThreadLocal.remove()方法）以规避内存泄漏的风险。





### 如何缩减APK包大小？





1 Memory Monitor

观察内存使用情况

2 Allocation Tracker

跟踪内存分配情况

3 Heap Dump

观察内存使用情况

4 MAT  

Memory Analysis Tool

分析堆存储文件 hprof，DDMS 或者 Memory Monitor 生成

5 Dominator Tree

分析对象实例引用关系

6 Histogram

分析类的引用关系

7 OQL

Object Query Language 

对象查询语言，用于查询当前内存中满足指定条件的所有对象

8 LeakCanary

基于 MAT ，在代码加入监测内存泄漏语句



#### Fragment如果在Adapter中使用应该如何解耦？





#### 设计一个音乐播放界面，你会如何实现，用到那些类，如何设计，如何定义接口，如何与后台交互，如何缓存与下载，如何优化？



#### 从0设计一款App整体架构，如何去做？





#### 项目框架里有没有Base类，BaseActivity和BaseFragment这种封装导致的问题，以及解决方法？





#### 实现一个库，完成日志的实时上报和延迟上报两种功能，该从哪些方面考虑？





安全问题

Android与服务器交互的方式中的对称加密和非对称加密是什么?

对于Android 的安全问题，你知道多少





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


















