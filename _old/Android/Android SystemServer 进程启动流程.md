

## SystemServer 进程

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



### ZygoteInit#startSystemServer 



由于 Zygote fork 自身创建了 SystemServer,所以同时那个 Server Socket 也复制过来了，没有用处需要通过 zygoteServer.closeServerSocket 先关掉



#### ZygoteInit#handleSystemServerProcess 

##### 创建 PathClassLoader

##### ZygoteInit#zygoteInit

###### ZygoteInit#nativeZygoteInit

- 调用 native 方法启动原生线程池，是 AndroidRuntime 里的 
- 内部掉用子类 AppRuntime 的onZygoteInit
- 内部调用 startThreadPool
- 内部有个mThreadPoolStarted 布尔标志位，确保 Binder 线程池只能启动一次
- 如果第一次启动调用 spawnPooledThread 来创建线程池的第一个线程即主线程
- PoolThread 走 run 方法
- 然后 joinThreadPool 将创建的线程加入到 Binder 线程池中

###### RuntimeInit#applicationInit

- 调用 RuntimeInit#invokeStaticMain 方法
- 内部通过 Class#forName 反射得到 SystemServer 类，name 就是上文提到的 com.android.server.SystemServer 类
- 找到 SystemServer  的 main 方法备用，这里没有之间调用  SystemServer#main 而是通过抛出一个指定异常 Zygote$MethodAndArgsCaller 让 ZygoteInit#main 方法去捕获该异常
- ZygoteInit#main 捕获到异常后，caller.run 间接执行 SystemServer#main，run 内部其实就是 invoke 动态调用



### SystemServer#main

- 内部直接调用了 SystemServer 对象的 run 方法

- 内部 Looper#prepareMainLooper
- System#loadLibrary("android_servers") 加载 so 库
- createSystemContext
- 实例化 SystemServiceManager
- startBootstrapServices ，启动 AMS、PowerManagerService、PackageManagerService 等服务
- startCoreServices，启动 BatteryService、BluetoothService、NotificationService 等服务
- startOtherServices，启动 CameraService、WMS、AudioService 等服务





为什么 SystemServer 不在 init 进程里直接启动而是在 Zygote 里？



### 总结

ZygoteInit#main 间接调用了 SystemServer#main

启动 Binder 线程池，可以与其他进程通信

创建 SystemServiceManager，可用于对系统服务的创建、启动等操作

 bootstrap core other 启动一系列系统服务

