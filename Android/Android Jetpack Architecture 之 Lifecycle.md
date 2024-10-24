

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

 