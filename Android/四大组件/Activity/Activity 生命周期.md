# 1 完整生命周期
- 6个
```
onCreate —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestory
```
![Activity生命周期](https://img-blog.csdnimg.cn/img_convert/992085ea3836734bf8b438f4e4a9067f.gif)


- 1所有Activity生命周期方法的实现都必须先调用其父类版本。

- 2由于Activity经常会暂停和恢复之间切换，所以onResume和onPause这两个方法应当是轻量级的。

- 3在系统再某种紧急情况下需要回收内存，onStop、onDestory可能不会被调用，因此需要在onPause中把需要长期保存的数据保存起来。

# 2 经典场景
场景 1：第一次启动 A


```
A：onCreate -> onStart -> onResume
```
场景 2：在 A 上启动 B    
```
A：onPause ->（A 被 B 覆盖）onStop -> （如果 A finish 了或被系统回收了）onDestory
B：onCreate -> onStart -> onResume（ B 会等到 A 的 onPause 运行后再执行，所以保存瞬时数据应该在 onPause 里，耗时操作需要开子线程处理）
```
场景 3：从 B 返回 A  
```
B：onPause -> onStop -> onDestory
A：
onRestart -> onStart -> onResume
// A finish 了或被系统回收了，则走
onCreate -> onStart -> onResume
```
场景 4：锁屏与解锁

```
//锁屏
onPause，不会走 onStop
//解锁
onResume
```
场景5  点击 HOME 键、来电
```
1、点击 Home 键、来电：onPause　->　onStop 
2、回到 App
onRestart ->  onStart　->　onResume  
// 前台 Activity finish 了或被系统回收了，则走
onCreate -> onStart -> onResume
```

场景6  横竖屏幕切换
- 1、不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次

- 2、设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次

- 3、设置Activity的android:configChanges="orientation|keyboardHidden"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法


# 3 生命周期该做的事


- onCreate()

  绑定布局，控件，初始化数据

声明 UI 元素，定义成员变量，配置 UI 等
尽量少做些事，避免程序启动太久而看不见界面
一旦onCreate() 操作完成，系统会迅速调用onStart() 与onResume()方法


- onDestoty()

需要将该activity彻底移除的信号时，系统会调用这个方法
大多数app并不需要实现onDestory()这个方法，由于局部类references会随activity的销毁而销毁
我们的activity应该在onPause(), onStop()中执行清除activity资源的操作
如果含有onCreate()中创建的后台线程，或是其他有可能导致内存泄漏的的资源，应在onDestory()中清理资源
在onCreate()中，直接调用finish()方法，系统会直接调用onDestory()方法，跳过其他的声明周期
//阿里推荐
不要在 Activity#onDestroy()内执行释放资源的工作，例如一些工作线程的
销毁和停止，因为 onDestroy()执行的时机可能较晚。可根据实际需要，在
Activity#onPause()/onStop()中结合 isFinishing()的判断来执行。

- onPause()

停止动画或其他运行的的操作，那些都会导致cpu浪费
提交用户离开时期保存的内容(例如邮箱草稿)
释放系统资源，Camera,Broadcast Receiver,sensors(传感器，比如GPS) ，以及任何其他影响电量的资源
不应该保存用户数据到永久存储上 (File或Db中)
尽量减少onPause() 中的工作量避免切换到下一个activity变得缓慢
//阿里推荐
当前Activity的onPause方法执行结束后才会执行下一个Activity的onCreate
方法，所以在 onPause 方法中不适合做耗时较长的工作，这会影响到页面之间的跳
转效率。
在 Activity.onPause()或 Activity.onStop()回调中，关闭当前 activity 正在执
行的的动画。

- onResume()

恢复activity时，应该初始化那些onPause() 释放掉的组件
执行那些activity每次进入resume state 都需要进行初始化的动作


- onStop()

应该释放那些不再需要的所有资源，避免内存泄漏，onStop() 后系统会在需要内存空间时摧毁它的实例
使用onStop()来执行那些CPU intensive的shut-down操作，例如往数据库写信息
系统会保存布局中视图的当前状态，例如，可以保存EditText中的文本内容，停止通过Service定时更新UI上的数据的Service


- onStart()

在onStop()中里面做了那些清除的操作，就应该做instant中把那些清除掉的资源重新创建出来



4 什么时候只会走 onPause，不会走 onStop

- 锁屏
- 打开一个透明主题的页面或者对话主题的页面



5 Activity不执行onDestory()

下拉状态栏时Activity的生命周期





#### 异常情况下的生命周期

内存不足被系统回收

数据如何保存和恢复





统计Activity的工作时间



启动一个其它应用的Activity的生命周期分析

 