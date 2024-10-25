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

