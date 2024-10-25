# Android 系统启动流程
- Android 从开机启动到成功运行完 Launcher 桌面启动器的整个流程

## BootLoader
- 引导程序，是在 Android 系统启动前的一个小程序，主要作用就是把系统引导运行起来，类似 PC 电脑里 BIOS 的概念

## Init 进程
- 最主要的作用就是 fork （复制）出 Zygote 进程
- 会创建 ServiceManager 进程？

## Zygote 进程
- 分裂器，Zygote 进程一分为二（进程分裂），一个继续作为 Zygote 进程，另一个在执行完相关初始化代码后就变成新进程，而这个进程也顺势拥有了启动好的 Java 虚拟机，注册好的 JNI 方法，这种方式能够大大提高新进程的启动速度
- 1 C++ 层的 app_main.cpp 里会通过 startVM 函数去启动 Java 虚拟机，然后通过 startReg 函数去注册 JNI 方法
- 2 然后会通过 JNI 方式调用 Java 的方法，就是 ZygoteInit.java 类的 main 方法，main 方法里会预加载一些资源，然后会去创建一个 ZygoteServer（实际上是一个 Socket 服务，是用来进行跨进程间通信的），接着去 fork 出 SystemServer 进程，然后 Zygote 进程继续进入一个 Handler/Looper 里类似的 loop 循环状态，一直等待接收 AMS 可能会创建进程的消息，假设收到消息了就同样会进行进程分裂操作

## SystemServer
SystemServer 是用来创建 AMS，PMS，WMS 等进程服务
- 1 ActivityThread.java 类里的 main 方法执行 Looper.prepareMainLooper，最后执行了 Looper.loop（在主线程中可以直接使用 Handler 的原因）
- 2 createSystemContext？
- 3 创建 SystemServiceManager，用于
- 4 startBootstrapServices 启动引导类服务（如 ActivityManagerService、PackageManagerService 等）
- 5 startCoreServices 启动核心类服务（如 SystemConfigService、BatteryService、WebViewUpdateService 等）
- 6 startOtherServices 启动其他类服务（如 WindowManagerService、并且利用 AMS 的 systemReady 方法启动了 Launcher 这个特殊的 App）

 
## Launcher
启动器（桌面、主屏幕），显然 Launcher App 和选中将要打开的 App 是运行在两个不同的进程中的，他们之间的通信是通过 Binder 来完成的
- Launcher 和 AMS 之间利用 Binder 通信
- 主要是查询现有 App 的列表（PMS）做展示，然后和 AMS 之间进行通信，而 AMS 又会和 Zygote 进程进行通信，Zygote 进程 fork 出了一个新进程来启动 App 进程


1 Launcher 发通知告诉 AMS 需要启动一个应用的根 Activity
```kotlin
//内部调用 intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK) 让 Activity 启动在新的任务栈中
//然后会调用了 Activity 的 startActivity 方法（ Launcher 继承自 Activity）
Launcher#startActivitySafely

//而 startActivity 方法内又调了 startActivityForResult 方法
Activity#startActivityForResult 

//startActivityForResult 调用了 Instrumentation 的 execStartActivity 方法，Instrumentation 主要是用来监控应用程序和系统交互的
Instrumentation#execStartActivity

//execStartActivity 里会调用 IActivityManager 的 startActivity 方法，传入 ActivityThread 对象和 IBinder 对象
ActivityManager.getService().startActivity

```
2 AMS 收到要启动根 Activity 的通知后，后续会发通知让 Launcher 进入 Pause 状态
ApplicationThread 接收到 AMS 发的消息后，调用 ActivityThread 的 sendMessage 方法，向 Launcher 发送一个 Activity Pause 的消息

3 Launcher Pause 后会通过 Process.start 去启动 ActivityThread
ActivityThread 就是那个有 Looper.prepareMainLooper 和 Looper.loop 方法的类，然后创建 Application


4 通过 ActivityThread$ApplicationThread#scheduleLaunchActivity 在目标进程中启动指定 Activity

5 后续会通过 ActivityThread#handleLaunchActivity 去做实际启动
Instrumentation#callActivityOnCreate
Activity#onCreate


调用Application的onCreate
执行Activity的onCreate
ActivityThread$H#sendMessage 
创建 Context
Instrumentation#newActivity
PakeageInfo#makeApplication


## App 进程
 应用程序进程



## 小结：
- 1 Init 进程 fork 出 Zygote 进程
- 2 Zygote 进程 fork 出 SystemServer 进程
- 3 SystemServer 进程创建出 AMS 、PMS 等服务
- 4 AMS 通知 Zygote 进程 fork 出 Launcher 进程
    - ActivityThread 
    - ApplicationThread
    - ActivityThread$ApplicationThread
- 5 AMS 创建出 Application，执行 Application 的 onCreate
- 6 AMS 创建出 Activity，执行 Activity 的 onCreate