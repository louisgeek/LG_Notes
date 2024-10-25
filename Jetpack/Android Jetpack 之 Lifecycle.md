

### 简介

lifecycle 解决2个问题
不让生命周期函数中写大量管理的组件代码，反过来，让组件自己去感知去管理，减少Activity/Fragment中代码，按组件更易于管理
无法保证 Activity/Fragment 停止后不继续执行启动，通过lifecycle得到的状态来进行先判断再执行逻辑，能够规避该问题

如果是 MVP ，那么 Presenter 可以实现 LifecycleObserver，就不需要生命周期函数主动调用了

### LifecycleRegistry

用于发送处理生命周期事件





### ProcessLifecycleOwner

以前用 registerActivityLifecycleCallbacks 进行处理 APP 处于前台还是后台判断，
需要自己维护 mStartedActivityCount ,mStoppedActivityCount，mResumedActivityCount，mPausedActivityCount，现在可以之间用 ProcessLifecycleOwner



- ProcessLifecycleOwnerInitializer 也是一个利用 ContentProvider 完成自动初始化的类





FastSafeIterableMap