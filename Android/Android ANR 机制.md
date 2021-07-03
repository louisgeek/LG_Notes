## Android ANR 机制



ANR 应用程序无响应

- Application Not Responding



https://developer.android.google.cn/topic/performance/vitals/anr?hl=zh_cn







### 会发生 ANR 的场景

- 5 秒内没有响应屏幕触摸事件或键盘输入事件
- 广播的 onReceive 方法耗时超过 10 秒
- 前台服务 20 秒内，后台服务 200 秒内没有执行完成
- ContentProvider的 publish 在 10 秒内没有执行完





