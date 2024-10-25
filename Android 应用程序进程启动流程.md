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
