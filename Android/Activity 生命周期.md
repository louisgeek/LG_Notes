## Activity 生命周期

- 官方文档https://developer.android.google.cn/guide/components/activities/activity-lifecycle

- 6个
```
onCreate —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestory
```
- onRestart 方法在 Activity 从不可见重新回到前台时调用

![Activity 生命周期](https://img-blog.csdnimg.cn/20210330222759521.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdWlzZ2Vlaw==,size_16,color_FFFFFF,t_70#pic_center)

- 所有生命周期方法重写实现必须调用其父类方法
- 由于 Activity 经常在暂停和恢复之间来回切换，所以 onResume 和 onPause 的逻辑应该是轻量级的
- 图中显示系统在某些情况下回收内存导致 onStop，onDestory 可能不被调用，因此可以根据实际情况在 onPause 中保存一些重要数据，耗时操作需要开子线程处理

## 经典场景

场景 1 第一次启动 Activity  A


```
//Activity A
onCreate -> onStart -> onResume
```
场景 2 在 Activity A 上启动 Activity B    
```
//Activity A
onPause ->(Activity A 被 Activity B 覆盖后走)onStop ->(如果 Activity A 自身 finish 或是被系统回收后走)onDestory
//Activity B
onCreate -> onStart ->(Activity B 会等到 Activity A 的 onPause 执行后走)onResume
```
- 如果 Activity  B 是完全透明或对话框主题的 ，那么 Activity A 就不继续走 onStop 
- onPause 不能进行耗时操作

场景 3 从 Activity B 返回到 Activity A  

```
//Activity B
onPause -> onStop -> onDestory
//Activity A
onRestart -> onStart -> onResume
//如果 Activity A 自身 finish 或是被系统回收后，则这么走
onCreate (重新创建) -> onStart -> onResume
```
场景 4 锁屏与解锁

```
//锁屏
onPause，不会走 onStop
//解锁
onResume
```
场景 5 点击 HOME 键、来电
```
//点击 Home 键、来电
onPause　->　onStop 
//回到 App
onRestart ->  onStart　->　onResume  
//前台 Activity finish 了或被系统回收了，则这么走
onCreate (重新创建) -> onStart -> onResume
```

## 生命周期该做的事

### onCreate

- 调用 setContentView 方法设置布局
- 定义成员变量，初始化数据
- 初始化视图、控件、UI 元素
- 配置 UI，将数据绑定到列表等 
- 将 Activity 与 ViewModel 相关联

应尽量减少 onCreate 的工作量，避免程序启动太久而看不见界面，可以通过 savedInstanceState 参数恢复一些状态，如果是第一次创建则为 null ，所以需要做判空处理



### onStart


- 如注册一个监听 UI 变化的广播
- 把在 onStop 中释放的资源重新创建回来



### onResume

- 把 onPause 中停止的操作恢复回来，如 Camera 预览
- 开始动画

意味着此时 Activity 位于 Activity 堆栈的顶部，获取了焦点

 

### onPause

- 释放系统资源，传感器(例如 GPS)手柄、Camera预览等
- 停止动画
- 停止视频播放

不应该在这里保存应用或用户数据、进行网络调用或执行数据库事务，即不适合做耗时较长的工作

地图导航页面一般不在这里释放，因为希望它仍然能够继续工作

应最大程度减少 onPause 的工作量避免 Activity 切换缓慢卡顿



### onStop

- 应该释放那些不再需要的资源

- 按需从精确位置更新切换到粗略位置更新
- 关闭那些 CPU 执行相对密集的操作
- 将用户内容(如邮箱草稿)保存到持久性存储空间
- 用户首选项持久性数据或数据库中的数据
- 停止通过 Service 定时更新 UI 上的数据的 Service

可以在这里保存应用或用户数据、进行网络调用或执行数据库事务



### onDestoty

- 应释放先前的回调(如 onStop )尚未释放的所有资源

不推荐在 onDestroy 里执行释放资源的工作，因为 onDestroy 执行的时机可能较晚，可根据实际需求在
onPause 或 onStop 中结合 isFinishing 判断来执行



## 异常情况下的生命周期

- 内存不足后导致 Activity 被系统回收，系统不会直接终止 Activity 以释放内存，而是会终止 Activity 所在的进程，系统不仅会销毁 Activity，还会销毁在该进程中运行的所有其他内容
- Configuration 配置发生了改变：横竖屏切换、系统语言改变、输入设备的改变、切换到多窗口模式( Android 7.0  Api 24 )
- 使用【设置】里的【应用管理器】来停止应用以终止进程

Activity 会先销毁再重建

### onSaveInstanceState

- super.onSaveInstanceState 里已经实现保存视图层次结构的状态的逻辑

- 保存有关 Activity 的 view hierarchy state 视图层次结构状态的瞬时信息(如输入框的值、列表滑动后停留的位置)，系统用于恢复先前状态的已保存数据称为实例状态，是存储在 Bundle 对象中的键值对集合，默认情况下，系统使用  Bundle 实例状态来保存 Activity 布局中每个 View 对象的相关信息，系统因系统限制（例如配置变更或内存压力）而销毁 Activity 时候，如果用户尝试回退到该 Activity，系统将使用一组描述 Activity 销毁时状态的已保存数据新建该 Activity 的实例，无需编写代码就能恢复布局状态为其先前的状态，

- Bundle 对象并不适合保留大量数据，在主线程中进行序列化和反序列化，会产生一定内存消耗

- 保存临时数据为主，保存简单轻量的界面状态，如果保存大量数据应该配合使用 ViewModel 进行处理

- 调用在 onStop 之前

- 主动调用 finish 方法和点击返回键的时候是不会调用该方法保存状态的

  

### onRestoreInstanceState

- super.onRestoreInstanceState 已经实现恢复视图层次结构的状态的逻辑

- 调用在 onStart 之后
- 可用于恢复一些 onSaveInstanceState 方法中保存的数据



### 横竖屏切换

- 不设置 Activity 的 android:configChanges 时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次

- 设置 Activity 的 android:configChanges="orientation" 时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次

- 设置 Activity 的 android:configChanges="orientation|keyboardHidden" 时，切屏不会重新调用各个生命周期，只会执行 onConfigurationChanged 方法

 

## 常见问题

1 如果在 onCreate 方法中直接调用 finish 方法，生命周期是怎样的？

系统会跳过其他生命周期直接调用 onDestory 方法，其实在任何一个生命周期调用 finish 方法都会跳过这之前的所有其他生命周期



2 什么时候只会走 onPause，而不会走 onStop ？

- 锁屏
- 打开一个完全透明或对话框主题的 Activity



3 Activity 在什么时候会出现不执行 onDestory 的情况？

- 主线程异常崩溃

- 应用被强杀



4 下拉状态栏时 Activity 的生命周期是什么？

不走任何生命周期，状态栏和 AlertDialog、Toast 等都是通过 WindowManager.addView 方法来显示的，对 Activity 的生命周期没有影响，另外可以通过 onWindowFocusChanged(boolean hasFocus) 监听状态栏，hasFocus 为 false 可以表示下拉状态，从而可以实现暂停视频等需求



5 启动一个其它应用的 Activity 的生命周期分析？



6 如何统计 Activity 的工作时间？