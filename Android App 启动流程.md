# Android App 启动流程
- Android App 根 Activity 启动流程

- Launcher 请求 AMS 过程
- AMS 到 ApplicationThread 的调用过程
- ActivityThread 启动 Activity

### Launcher 请求 AMS 过程

Launcher#startActivitySafely

- add flag FLAG_ACTIVITY_NEW_TASK 让 Activity 启动在新的任务栈中
- 调用 startActivity 方法

Activity#startActivity

又调了 Activity#startActivityForResult

Instrumentation#execStartActivity

ActivityManager.getService().startActivity

IBinder

小结

Launcher <- Binder -> AMS 

利用 Binder 通信

### AMS 到 ApplicationThread 的调用过程

ActivityManagerService#startActivity

ActivityManagerService#startActivityAsUser

内部校验调用者是否有权限

ActivityStarter#startActivityMagWait

ActivityStarter#startActivityLocked

ActivityStarter#startActivity

ActivityStarter#startActivityUnchecked

ActivityStackSupervisor#resumeFocusedStackTopActivityLocked

ActivityStack#topRunningActivityLocked

ActivityStack#resumeTopActivityUncheckedLocked

ActivityStack#resumeTopActivityInnerLocked

ActivityStackSupervisor#startSpecificActivityLocked

Ams#getProcessRecordLocked

ActivityStackSupervisor#realStartActivityLocked

ActivityThread$ApplicationThread#scheduleLaunchActivity

- 代码运行在 AMS 进程中，scheduleLaunchActivity 方法就表示在目标进程中启动指定 Activity

小结 

该过程关系如下

AMS <- Binder -> ApplicationThread

AMS 所在 SystemServer 进程

ApplicationThread 所在 ActivityThread ，而 ActivityThread 又在 应用程序进程

涉及  ProcessRecord 用于描述一个应用程序进程信息、ActivityRecord 用于描述一个 Acticvity 的信息

### ActivityThread 启动 Activity 的过程

ActivityThread$ApplicationThread#scheduleLaunchActivity、、

封装 ActivityClientRecord 通过 sendMessage 发送给 H（继承自 Handler），指令是 H.LAUNCH_ACTIVITY

ActivityThread$H#sendMessage 

ActivityThread#handleLaunchActivity

ActivityThread#performLaunchActivity

ActivityThread#handleResumeActivity

创建 Context

Instrumentation#newActivity

PakeageInfo#makeApplication

Activity#attach

Instrumentation#callActivityOnCreate

Activity#performCreate

Activity#onCreate

### 总结

Launcher <- Binder-> AMS 点击 APP 图标，请求 AMS 创建 APP 根 Activity

AMS -> Zygote -> ApplicationThread AMS 请求 Zygote 创建应用程序进程，Zygote 创建并启动应用程序进程

AMS <- Binder -> ApplicationThread 然后 AMS 请求 ApplicationThread  创建根 Activity 的行为可以继续进行






Zygote 启动 Server Socket 来监听 AMS 发送的请求，让 Zygote 利用 fork 自身方式创建应用程序进程



### AMS 发送启动应用程序进程的请求

ActivityManagerService#startProcessLocked

定义了一个 entryPoint="android.app.ActivityThread" 传入方法

Process#start

直接调用了

ZygoteProcess#start

也是直接调用了

ZygoteProcess#startViaZygote 

准备一些应用进程启动参数 argsForZygote 传入下面这个方法，同时也传入了一个 openZygoteSocketIfNeeded 方法返回的值的参数

- ZygoteProcess$ZygoteState#connect 
- 处理 mSocket 和 mSecondarySocket ,返回 primaryZygoteState 和 secondaryZygoteState

ZygoteProcess#zygoteSendArgsAndGetResult

将 argsForZygote 通过字节流写入到 ZygoteProcess$ZygoteState，并且构造 Process$ProcessStartResult 后返回







### Zygote 接收请求后创建应用程序进程

见【Android Zygote 进程启动流程】
