

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






- Jetpack Architecture 架构组件中提供用来更方便地管理生命周期事件的组件
- 是很典型的观察者模式的实践

在生命周期变动时，把之前需要在activity或者fragment需要处理的各个事件抽离出来，集中到一个地方进行事件处理，方便统一管理

LifecycleObserver 接口

```kotln
/**
 * Marks a class as a LifecycleObserver. Don't use this interface directly. Instead implement either
 * [DefaultLifecycleObserver] or [LifecycleEventObserver] to be notified about
 * lifecycle events.
 *
 * @see Lifecycle Lifecycle - for samples and usage patterns.
 */
public interface LifecycleObserver
```


DefaultLifecycleObserver
```kotlin
/**
 * Callback interface for listening to [LifecycleOwner] state changes.
 * If a class implements both this interface and [LifecycleEventObserver], then
 * methods of `DefaultLifecycleObserver` will be called first, and then followed by the call
 * of [LifecycleEventObserver.onStateChanged]
 *
 *
 * If a class implements this interface and in the same time uses [OnLifecycleEvent], then
 * annotations will be ignored.
 */
public interface DefaultLifecycleObserver : LifecycleObserver {
    /**
     * Notifies that `ON_CREATE` event occurred.
     *
     *
     * This method will be called after the [LifecycleOwner]'s `onCreate`
     * method returns.
     *
     * @param owner the component, whose state was changed
     */
    public fun onCreate(owner: LifecycleOwner) {}

    /**
     * Notifies that `ON_START` event occurred.
     *
     *
     * This method will be called after the [LifecycleOwner]'s `onStart` method returns.
     *
     * @param owner the component, whose state was changed
     */
    public fun onStart(owner: LifecycleOwner) {}

    /**
     * Notifies that `ON_RESUME` event occurred.
     *
     *
     * This method will be called after the [LifecycleOwner]'s `onResume`
     * method returns.
     *
     * @param owner the component, whose state was changed
     */
    public fun onResume(owner: LifecycleOwner) {}

    /**
     * Notifies that `ON_PAUSE` event occurred.
     *
     *
     * This method will be called before the [LifecycleOwner]'s `onPause` method
     * is called.
     *
     * @param owner the component, whose state was changed
     */
    public fun onPause(owner: LifecycleOwner) {}

    /**
     * Notifies that `ON_STOP` event occurred.
     *
     *
     * This method will be called before the [LifecycleOwner]'s `onStop` method
     * is called.
     *
     * @param owner the component, whose state was changed
     */
    public fun onStop(owner: LifecycleOwner) {}

    /**
     * Notifies that `ON_DESTROY` event occurred.
     *
     *
     * This method will be called before the [LifecycleOwner]'s `onDestroy` method
     * is called.
     *
     * @param owner the component, whose state was changed
     */
    public fun onDestroy(owner: LifecycleOwner) {}
}
```


LifecycleOwner 生命周期拥有者，比如 ComponentActivity 和 Fragment 都实现了这个接口，还有 lifecycle-service 里的 LifecycleService

```kotlin
/**
 * A class that has an Android lifecycle. These events can be used by custom components to
 * handle lifecycle changes without implementing any code inside the Activity or the Fragment.
 *
 * @see Lifecycle
 * @see ViewTreeLifecycleOwner
 */
public interface LifecycleOwner {
    /**
     * Returns the Lifecycle of the provider.
     *
     * @return The lifecycle of the provider.
     */
    public val lifecycle: Lifecycle
}

```

 