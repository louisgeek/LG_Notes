# Android 系统启动流程
- Android 从开机启动到成功运行完 Launcher 桌面启动器的整个流程

## BootLoader 启动引导
- 接通电源，按下启动按钮，ROM 中的引导芯片代码开始执行，会加载 BootLoader 到 RAM 中执行
- 引导程序，是在 Android 操作系统启动前的一个小程序，主要作用就是把操作系统引导运行起来，类似 PC 电脑里 BIOS 的概念
- 引导完后 Linux 内核启动进行加载驱动和启动系统服务等系统设置，查找 init.rc 文件，启动 Init 进程

## Init 进程
- 最主要的作用就是 fork 复制出 Zygote 进程，根据配置文件启动 Zygote 进程等

## Zygote 进程
- 分裂器，Zygote 进程一分为二（进程分裂），一个继续作为 Zygote 进程，另一个在执行完相关初始化代码后就变成新进程，而这个进程也顺势拥有了启动好的 Java 虚拟机，注册好的 JNI 方法等，这种方式能够大大提高新进程的启动速度
- 1 C++ 层的 app_main.cpp 里会通过 startVM 函数去启动 Java 虚拟机，然后通过 startReg 函数去注册 JNI 方法
- 2 然后会通过 JNI 方式调用 Java 的方法，就是 ZygoteInit.java 类的 main 方法，main 方法里会预加载一些资源，然后会去创建一个 ZygoteServer（实际上是一个 Socket 服务，是用来进行跨进程间通信的），接着去 fork 出 SystemServer 进程，然后 Zygote 进程继续进入一个 Handler/Looper 里类似的 loop 循环状态，一直等待接收 ActivityManagerService 可能会创建新应用进程的消息，假设收到消息了就同样会进行进程分裂操作，Zygote 会 fork 出一个子进程来运行新的 App 应用

## SystemServer 进程
- SystemServer 是用来创建 ActivityManagerService 和 WindowManagerService 等进程服务
    - 1 ActivityThread.java 类里的 main 方法执行 Looper.prepareMainLooper，最后执行了 Looper.loop（在主线程中可以直接使用 Handler 的原因）
    - 2 通过 createSystemContext 来创建一个 ContextImpl 系统上下文
    - 3 创建 SystemServiceManager，用于管理服务的启动和生命周期
    - 4 startBootstrapServices 启动引导类服务（如 ActivityManagerService、PackageManagerService 等）
    - 5 startCoreServices 启动核心类服务（如 SystemConfigService、BatteryService、WebViewUpdateService 等）
    - 6 startOtherServices 启动其他类服务（如 WindowManagerService、并且利用 AMS 的 systemReady 方法去启动 Launcher 这个特殊的 App）

## Launcher
- 启动器（桌面、主屏幕），显然 Launcher App 和点击图标选中将要打开的 App 是运行在两个不同的进程中的，他们之间的通信是通过 Binder IPC 进程间来完成的，Launcher App 主要是查询安卓系统中已安装应用的列表（PMS）做展示，然后和 AMS 之间进行通信，而 AMS 又会和 Zygote 进程进行通信，Zygote 进程 fork 出了一个新进程来启动 App 进程
    - 1 Launcher 发通知告诉 AMS 需要启动一个新应用的根 Activity
```
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
    - 2 AMS 收到要启动根 Activity 的通知后，后续会发通知让 Launcher 进入 Pause 状态
ApplicationThread 接收到 AMS 发的消息后，调用 ActivityThread 的 sendMessage 方法，向 Launcher 发送一个 Activity Pause 的消息
    - 3 Launcher Pause 后会通过 Process.start 去启动 ActivityThread，然后创建 Application
    - 4 通过 ActivityThread$ApplicationThread#scheduleLaunchActivity 在目标进程中启动指定 Activity
    - 5 后续会通过 ActivityThread#handleLaunchActivity 去做实际启动
Instrumentation#callActivityOnCreate
Activity#onCreate

调用 Application 的 onCreate
执行 Activity 的 onCreate
ActivityThread$H#sendMessage 
创建 Context
Instrumentation#newActivity
PakeageInfo#makeApplication


## App 进程
 应用程序进程



## 小结
- 1 Init 进程 fork 出 Zygote 进程
- 2 Zygote 进程 fork 出 SystemServer 进程
- 3 SystemServer 进程创建出 AMS 、PMS 等服务
- 4 AMS 通知 Zygote 进程 fork 出 Launcher 进程
    - ActivityThread 
    - ApplicationThread
    - ActivityThread$ApplicationThread
- 5 AMS 创建出 Application，执行 Application 的 onCreate
- 6 AMS 创建出 Activity，执行 Activity 的 onCreate







Launcher 

- 启动器，是一个应用程序，可以供用户启动 APP
- 安卓系统的桌面，可以显示 APP 快捷图标和桌面组件
- Launcher 的入口是 AMS 的 systemReady 方法



SystemServer#startBootstrapServices

- 完成了启动 AMS 和 PackageManagerService

SystemServer#startOtherServices

- ActivityManagerService#systemReady 传入一个 Callback
- ActivityStackSupervisor#resumeFocusedStackTopActivityLocked
- ActivityStack#resumeTopActivityUncheckedLocked
- ActivityStack#resumeTopActivityInnerLocked
- ActivityStackSupervisor#resumeHomeStackTask
- ActivityManagerService#startHomeActivityLocked
- ActivityStarter#startHomeActivityLocked
- mSupervisor.moveHomeStackTaskToTop







Launcher 中图标的显示过程

Launcher#onCreate

```
LauncherAppState app = LauncherAppState.getInstance();
...
mModel = app.setLauncher(this);
...
mModel.startLoader(xxx)
```



- LauncherAppState#setLauncher(this)

- ```java
  LauncherModel setLauncher(Launcher launcher){
  ...
  mModel.initialize(launcher); //传入 launcher
  return mModel;
  }
  ```

- LauncherModel#initialize

  ```
  public void initialize(Callbacks callbacks){
  mCallbacks  = new WeakReference<Callbacks>(callbacks);//封装成 WeakReference
  }
  ```

  

  LauncherModel#startLoader

  ```
  mLoaderTask = new LoaderTask(mAPP,isLaunching);// LoaderTask 实现了 Runnable
  mWorker.post(mLoaderTask);// mWorker 是一个 Handler
  ```

- LauncherModel$LoaderTask#run

  - LoaderTask#loadWorkSpace
  - LoaderTask#bindWorkSpace
  - LauncherModel#loadAllApps

LauncherModel#loadAllApps

- callbacks.bindAllApplications(added)

  

  Launcher#bindAllApplications

  - AllAppsContainerView#setApps 把 List<AppInfo> 类型的 apps 设置到 AlphabeticalAppsList 上
  - AllAppsContainerView#onFinishInflate 

  - 内部 findViewById 找到一个 apps_list_view 的 AllAppsRecyclerView
    - 然后填充数据、setAdapter 把所有图标显示到列表上

  
