

## Lifecycle

- Lifecycle 组件主要是解决了 2 个问题
  不让生命周期函数中写大量管理的组件代码，反过来，让组件自己去感知去管理，减少 Activity/Fragment 中代码，按组件划分更易于管理
  无法保证 Activity/Fragment 停止后不继续执行启动，通过 Lifecycle 得到的状态来进行先判断再执行逻辑，能够规避该问题
- 如果是 MVP 架构，那么 Presenter 可以实现 LifecycleObserver，就不需要生命周期函数主动调用了
- 是一个接口



## LifecycleRegistry

- androidx.activity.ComponentActivity#getLifecycle 返回的就是 androidx.activity.ComponentActivity#mLifecycleRegistry 变量
- androidx.lifecycle.LifecycleRegistry 继承 androidx.lifecycle.Lifecycle 抽象类
- androidx.lifecycle.LifecycleRegistry 构造函数传入一个 androidx.lifecycle.LifecycleOwner

用于发送处理生命周期事件







## FastSafeIterableMap







## ViewModelStore







## Fragment 作为 LifecycleOwner 的问题









## androidx.lifecycle.ProcessLifecycleOwner

- ```groovy
  
  implementation 'androidx.lifecycle:lifecycle-process:2.3.1'
  ```

- ProcessLifecycleOwner 实现了 androidx.lifecycle.LifecycleOwner

 

以前用 Application#registerActivityLifecycleCallbacks 的方式来进行判断 APP 处于前台还是后台的，但是该方案需要自行维护 mStartedActivityCount，mStoppedActivityCount，ResumedActivityCount，mPausedActivityCount 等变量去实现相关功能，相对来说麻烦，现在可以直接用 ProcessLifecycleOwner 来实现，它的实现原理基本和原先自行实现的方式思路一致，也是通过 registerActivityLifecycleCallbacks 和一些变量维护进行逻辑实现

- 其中 ProcessLifecycleOwnerInitializer 继承了 ContentProvider 类，也是一个利用 ContentProvider 机制完成自动初始化的类





## androidx.lifecycle.ReportFragment

- 专门用于分发生命周期事件的 Fragment

