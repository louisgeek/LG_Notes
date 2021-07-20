## Android ANR 机制



ANR 应用无响应

- Application Not Responding



如果 Android 应用的界面线程处于阻塞状态的时间过长，如果应用位于前台，系统会向用户显示一个 ANR 对话框会为用户提供强行退出应用的选项



- 在主线程进行耗时的 I/O 的操作

- 在主线程进行长时间的计算

- 主线程在对另一个进程进行同步 binder 调用，而后者需要很长时间才能返回。

- 主线程处于阻塞状态，为发生在另一个线程上的长操作等待同步的块。

- 主线程在进程中或通过 binder 调用与另一个线程之间发生死锁。主线程不只是在等待长操作执行完毕，而且处于死锁状态。







https://developer.android.google.cn/topic/performance/vitals/anr?hl=zh_cn







### 会发生 ANR 的场景

- 5 秒内没有响应屏幕触摸事件或按钮、键盘输入事件
- BroadcastReceiver 广播的 onReceive 方法耗时超过 10 秒
- 前台服务 20 秒内，后台服务 200 秒内没有执行完成
- ContentProvider的 publish 在 10 秒内没有执行完







StrictMode

#### 启用后台 ANR 对话框

**开发者选项**中启用了**显示所有 ANR** 



#### TraceView

Android Device Monitor



```
/data/anr/traces.txt
//新版
/data/anr/anr_*
```





如果主线程无法继续执行，则它处于 `BLOCKED` 状态，并且无法响应事件。该状态在 Android Device Monitor 中会显示为“Monitor”或“Wait”





### 死锁

线程进入等待状态时会发生死锁，因为所需资源由另一个线程持有，而该线程也在等待第一个线程持有的资源。如果应用的主线程处于这种情况，很可能会发生 ANR







 BroadcastReceiver 中 onReceive 代码要尽量减少耗时，耗时建议使用 IntentService 处理











[ANR监测机制 - 简书 (jianshu.com)](https://www.jianshu.com/p/ad1a84b6ec69)

