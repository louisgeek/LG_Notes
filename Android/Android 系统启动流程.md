### Android 系统启动流程

```
接通电源，按下启动按钮
ROM 中引导芯片代码开始执行
加载 BootLoader 到 RAM 执行
BootLoader 负责拉起系统 OS
Linux 内核启动，完成系统设置，查找 init.rc 文件，启动 init 进程
初始化和启动属性服务，启动 Zygote 进程
创建 Java 虚拟机并注册 JNI 方法，创建 Server Socket，启动 SystemServer 进程
创建 Binder 线程池和 SystemServiceManager,启动各种系统服务
启动 Launcher ，显示已安装 APP 列表
```



- 接通电源，按下启动按钮
ROM 中引导芯片的代码开始执行
加载 BootLoader 引导程序到 RAM 中后开始执行



- BootLoader 
是 Android 系统开始运行前的一个引导程序，我的理解是类似 Windows 的 BIOS (不过安卓有 Recovery 的概念)
负责拉起操作系统



- Linux 内核启动
进行一些列系统设置，完成内核加载
从系统文件中查找 init.rc 文件，启动 init 进程



- init 进程启动
  进行初始化和启动属性服务
  启动 Zygote 进程

  

- Zygote 孵化器进程启动



- SystemServer 进程启动
用于创建系统服务,如 AMS、WMS、PMS，ActivtyManagerService 启动



- Launcher 启动