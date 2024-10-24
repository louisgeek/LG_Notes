- Launcher 请求 AMS 过程
- AMS 到 ApplicationThread 的调用过程
- ActivityThread 启动 Activity


 



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



