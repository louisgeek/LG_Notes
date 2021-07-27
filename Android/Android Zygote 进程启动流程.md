### Zygote 进程

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
进入到 ZygoteInit#main 方法里
- ZygoteServer#registerServerSocket 创建名为"zygote"的 Server Socket
- ZygoteInit#preload 预加载类和资源
- ZygoteInit#startSystemServer，对应类名 com.android.server.SystemServer
- ZygoteServer#runSelectLoop 等待 AMS 请求创建新的应用程序进程
```





### main 函数

- 由于是 fork 自身来创建子线程的，所以Zygote 进程和它的子进程都会走这个 main 函数

判断是否带 --zygote 参数来判断是否运行在 Zygote 进程中，标记 zygote 布尔值为 true

判断是否带 --start-system-server 参数来判断是否运行在 SystemServer 进程中，标记 startSystemServer 布尔值为 true

继续，如果 zygote 为 true

那么调用 AppRuntime 的 start 函数

```
//推测是 Android系统里 Java 写的类
runtime.start("com.android.internal.os.ZygoteInit",args,zygote)
```

start 函数

先找到 ZygoteInit 的 main 方法

然后通过 JNI 方法 CallStaticVoidMethod 调用这个 Java 类 ZygoteInit 里的 main 方法

### ZygoteInit#main

- 从现在开始进入 Java 层，Zygote 开创了 Java 层

#### ZygoteServer#registerServerSocket 

- 创建一个名称是 zygote 的 Server 端 Socket

```
//socket 全名 ANDROID_SOCKET_zygote ,ANDROID_SOCKET_ 是固定前缀
mServerSocket = new LocalServerSocket(fd);
```

#### 通过 preload 方法预加载类和资源



#### ZygoteInit#startSystemServer 

- 按需启动系统服务

  ```
  //服务名 system_server，对应类名路径如下
  com.android.server.SystemServer
  ```
  
  通过 ZygoteConnection.Arguments 和 ZygoteConnection 的方法处理好一系列应用进程启动参数的配置
  
  接着用 Zygote#forkSystemServer 创建子进程（内部调用 nativeForkSystemServer 函数，最终也是通过 fork 函数进行子进程的创建），就是 SystemServer 进程，然后返回一个 pid
  
  判断 pid == 0 ，表示代码运行在子进程中，接着内部就调用 ZygoteInit#handleSystemServerProcess 方法
  
  

#### ZygoteServer#runSelectLoop 

- 等待 AMS 请求创建新的应用程序进程

内部有个 while 无限循环，得到 ZygoteConnection 调用其 runOnce 方法，传入 ZygoteServer 作为参数

readArgumentList 得到字符串数组格式的应用程序进程的启动参数

传入 ZygoteConnection.Arguments 构造得到 Arguments 

然后 Zygote#forkAndSpecialize 创建应用程序进程，然后返回一个 pid

判断 pid == 0 ，表示代码运行在子进程中，接着内部就调用 ZygoteConnection#handleChildProc 方法来处理应用程序进程，不等于 0 则走 handleParentProc

ZygoteInit#handleChildProc 调用了 ZygoteInit#zygoteInit

从这开始就和 【Android SystemServer 进程启动流程】后面基本一致了，区别是它通抛异常间接启动了前文定义的 entryPoint="android.app.ActivityThread" ，而不是之前的 com.android.server.SystemServer 了，这一段流程两者原理一致，也是不直接调用 ActivityThread#main



为什么需要单独的 Zogote 进程去孵化应用程序进程，而不是用 SystemServer 去处理？





为啥不采用 Binder 机制进行 IPC 通信？





### 总结

Zygote 进程

- Android.mk 定义名字 "app_process "，进程启动后，Linux 系统下的 pctrl 系统调用 app_process ，将名字改成 "zygote"
- init 进程启动时创建了 Zygote 进程
- Zygote 进程启动时会创建 ART（Dalvik）
- 通过 AndroidRuntime.cpp#start 函数启动 Zygote 进程(ZygoteInit)
- 创建 Java 虚拟机，并为 Java 虚拟机注册 JNI 方法（Java Rumtime 被加载到进程中并注册一系列 Android 核心类的 JNI）
- 通过 JNI 调用 ZygoteInit#main 方法，ZygoteInit#main 是 Java 框架层面的入口
- 利用 registerServerSocket 创建 Server Socket ，runSelectLoop 等待 AMS 请求，预加载类和资源
- 并按需利用 fork 自身创建应用程序进程和 SystemServer 进程（而通过 fork 都会将 Zygote 里的虚拟机实例复制到新的 App进程 里面去，从而使每个 App进程 都有一个独立的虚拟机实例，而且会和 Zygote 进程一起共享 Java Rumtime，也就是可以将 XposedBridge.jar 这个Jar加载到每个App进程 中去）
- ps：ZygoteInit#main 间接调用了 ActivityThread#main

