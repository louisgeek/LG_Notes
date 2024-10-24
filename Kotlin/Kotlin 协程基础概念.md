
# Kotlin 协程基础概念


## CoroutineScope 协程作用域
- 所有协程都必须在一个作用域内运行，一个 CoroutineScope 可以管理一个或多个相关的协程
- 一般可以调用作用域的 cancel 方法取消作用域，已取消的作用域无法再创建协程，所以一般销毁的地方调用，比如在 Activity 的 onDestroy 回调里
- CoroutineScope 重载了 plus 操作符，所以可以用 + 号组合

### 1 GlobalScope
 - GlobalScope 是一个 object 类，拥有一个 coroutineContext 是 EmptyCoroutineContext
 - 官方不推荐日常使用，GlobalScope.cancel 会有问题
```kotlin
public object GlobalScope : CoroutineScope {
    /**
     * Returns [EmptyCoroutineContext].
     */
    override val coroutineContext: CoroutineContext
        get() = EmptyCoroutineContext
}
```

### 2 CoroutineScope(dispatchers)
- 比如 CoroutineScope(Dispatchers.Main)
- CoroutineScope 方法（ps:方法名为啥用大写？）会返回一个 CoroutineScope 接口的实现类 ContextScope 的对象


### 3 MainScope()
- MainScope 方法（ps:方法名为啥用大写？）会返回一个 CoroutineScope 接口的实现类 ContextScope 的对象
- SupervisorJob 方法返回的是一种特殊的 Job，当子协程发生异常被取消不会同时取消该 Job 的其它子协程
- Dispatchers.Main 代表在 Android 主线程

```kotlin
public fun MainScope(): CoroutineScope = ContextScope(SupervisorJob() + Dispatchers.Main)
```

### 4 coroutineScope （比如 ComponentActivity 和 Fragment 里的 Lifecycle）
- coroutineScope 是 Lifecycle 的扩展属性
- Dispatchers.Main.immediate 是一种特殊的 Dispatchers.Main，当已经在主线程的情况下，它就不再用 post 到下一次消息循环中了，而是直接执行
 ```kotlin
 public val Lifecycle.coroutineScope: LifecycleCoroutineScope
    get() {
        while (true) {
            val existing = internalScopeRef.get() as LifecycleCoroutineScopeImpl?
            if (existing != null) {
                return existing
            }
            val newScope = LifecycleCoroutineScopeImpl(
                this,
                SupervisorJob() + Dispatchers.Main.immediate
            )
            if (internalScopeRef.compareAndSet(null, newScope)) {
                newScope.register()
                return newScope
            }
        }
    }
```

### 5 lifecycleScope
- LifecycleOwner 常用的比如 ComponentActivity 和 Fragment
- lifecycleScope 就是 lifecycle.coroutineScope

```kotlin
public val LifecycleOwner.lifecycleScope: LifecycleCoroutineScope
    get() = lifecycle.coroutineScope
 ```

### 6 viewModelScope
- viewModelScope 是 ViewModel 的扩展属性

```kotlin
public val ViewModel.viewModelScope: CoroutineScope
    get() {
        val scope: CoroutineScope? = this.getTag(JOB_KEY)
        if (scope != null) {
            return scope
        }
        return setTagIfAbsent(
            JOB_KEY,
            CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
        )
    }
```

## CoroutineContext 协程上下文
- CoroutineContext 是一个接口， CoroutineContext.Element 接口继承了 CoroutineContext
- CoroutineContext 包含了用户定义的一些各种不同元素的 Element 对象集合，包括管理了协程的生命周期，线程调度，异常处理等，Job 用于管理协程生命周期，Dispatchers 用于协程处理线程调度，将任务分派到适当的线程，CoroutineExceptionHandler 用于协程的异常处理，处理未捕获的异常，ContinuationInterceptor ，还有一个 CoroutineName 协程的名称，通常可用于调试时候的识别

- EmptyCoroutineContext 是一个 object 实现了 CoroutineContext 接口，一般作为默认的协程上下文来使用，比如 launch 或者 async 的第一个 CoroutineContext 参数默认就是 EmptyCoroutineContext
- CombinedContext 因为 CoroutineContext 也重载了 plus 操作符，而 CombinedContext 就是 + 号组合得到的

### CoroutineName
- CoroutineName data 数据类实现了 CoroutineContext.Element 接口，也就是算一个 CoroutineContext

### Job 任务
- Job 是一个接口，继承了 CoroutineContext.Element 接口，也就是算一个 CoroutineContext

### Dispatchers 协程调度器
- 抽象类 CoroutineDispatcher 实现了 CoroutineContext.Element 接口，也就是算一个 CoroutineContext
- Dispatchers 是一个 object 类
- 调度器确定了协程在哪个线程或哪些线程上执行。它可以将协程限制在一个特定的线程中执行，或将它分派到一个线程池，也或者让它不受限地运行

Dispatchers 里的 CoroutineDispatcher 类型
- 1 Dispatchers.Default 默认，对应 DefaultScheduler，为 JVM 共享线程池,$schedulerName-worker-${index},默认前缀是 DEFAULT_SCHEDULER_NAME = "DefaultDispatcher"，比如 DefaultDispatcher-worker-1，比如用于图片裁剪、二维码计算、Json解析操作等
- 2 Dispatchers.Main  对应 MainCoroutineDispatcher，Android 主线程
- 3 Dispatchers.IO  对应 DefaultIoScheduler，IO 线程池，默认为 64 个线程，比如用于网络请求，文件读写，数据库操作等
- 4 Dispatchers.Unconfined  对应 kotlinx.coroutines.Unconfined，不限制，当前 CoroutineScope 的线程策略

- ExecutorService 的 asCoroutineDispatcher 扩展方法可以把线程池作为协程调度器，比如 Executors.newSingleThreadExecutor().asCoroutineDispatcher()
- Handler 的 asCoroutineDispatcher 扩展方法可以把 Handler 所在线程作为协程调度器，比如 Handler(Looper.getMainLooper()).asCoroutineDispatcher()

ps:可以看到它叫 Dispatchers 而不是 Threads，比如 Threads.Main，Threads.IO 等等
### CoroutineExceptionHandler 协程异常处理器
- CoroutineExceptionHandler 是一个接口，继承了 CoroutineContext.Element 接口，也就是算一个 CoroutineContext
- 统一给父协程定义一个 CoroutineExceptionHandler，如果一个子协程异常了，其他子协程也不需要继续的话就采用 coroutineScope，默认的 Job 就是这种表现，如果一个子协程异常了，其他子协程也需要继续的话就采用 supervisorScope，或者创建协程作用域的时候传入带 SupervisorJob() 函数返回的特殊 Job 的协程上下文
- 子协程如果想单独处理异常就单独给子协程定义一个 CoroutineExceptionHandler

- 传入 exceptionHandler 方式和 supervisorScope
```kotlin
 val exceptionHandler = CoroutineExceptionHandler { coroutineContext, throwable ->
        Log.d("exceptionHandler", "${coroutineContext[CoroutineName].toString()} 处理异常 ：$throwable")
    }
    GlobalScope.launch(exceptionHandler) {
        supervisorScope {
            launch(CoroutineName("异常子协程")) {
                Log.d("${Thread.currentThread().name}", "我要开始抛异常了")
                throw NullPointerException("空指针异常")
            }
            for (index in 0..10) {
                launch(CoroutineName("子协程$index")) {
                    Log.d("${Thread.currentThread().name}正常执行", "$index")
                    if (index %3 == 0){
                        throw NullPointerException("子协程${index}空指针异常")
                    }
                }
            }
        }
    }
```

- 创建 CoroutineScope(SupervisorJob() + exceptionHandler) 方式
```kotlin
val exceptionHandler = CoroutineExceptionHandler { coroutineContext, throwable ->
        Log.d("exceptionHandler", "${coroutineContext[CoroutineName].toString()} 处理异常 ：$throwable")
    }
    val supervisorScope = CoroutineScope(SupervisorJob() + exceptionHandler)
    with(supervisorScope) {
        launch(CoroutineName("异常子协程")) {
            Log.d("${Thread.currentThread().name}", "我要开始抛异常了")
            throw NullPointerException("空指针异常")
        }
        for (index in 0..10) {
            launch(CoroutineName("子协程$index")) {
                Log.d("${Thread.currentThread().name}正常执行", "$index")
                if (index % 3 == 0) {
                    throw NullPointerException("子协程${index}空指针异常")
                }
            }
        }
    }
```


## CoroutineStart 协程启动模式
- CoroutineStart 是个枚举类，一般启动协程的第二个参数需要的就是它

- DEFAULT 默认启动模式，我们可以称之为饿汉启动模式，因为协程创建后立即开始调度，虽然是立即调度，但不是立即执行，有可能在执行前被取消
- LAZY 懒汉启动模式，启动后并不会有任何调度行为，直到我们需要它执行的时候才会产生调度。也就是说只有我们主动的调用 Job 的 start、join 或者 await 等函数时才会开始调度
- ATOMIC 一样也是在协程创建后立即开始调度，但是它和DEFAULT模式有一点不一样，通过ATOMIC模式启动的协程执行到第一个挂起点之前是不响应cancel 取消操作的，ATOMIC一定要涉及到协程挂起后cancel 取消操作的时候才有意义。
- UNDISPATCHED 协程在这种模式下会直接开始在当前线程下执行，直到运行到第一个挂起点。这听起来有点像 ATOMIC，不同之处在于 UNDISPATCHED 是不经过任何调度器就开始执行的。当然遇到挂起点之后的执行，将取决于挂起点本身的逻辑和协程上下文中的调度器
 


## 基础使用

### launch
- 是一个 CoroutineScope 接口的扩展方法，用于创建启动一个协程，不阻塞当前线程，并返回该新协程的 Job，内部有一个 join 的挂起函数,可以用于等待子协程运行完毕，这个 Job 也可用于控制协程的生命周期，比如在 Activity 的 onDestroy 回调里执行 job 的 cancel 方法取消该协程
- 第一个参数是 CoroutineContext 协程上下文，默认不设置传 EmptyCoroutineContext，此时如果 CoroutineScope 协程作用域里的 coroutineContext 未指定调度器的话，那么此时会采用调度器 Dispatchers.Default 处理（换句话说，如果传 EmptyCoroutineContext  时候，那么此时的调度器是 CoroutineScope 里的coroutineContext 决定的）
- 不阻塞线程
- 可全局创建
- 任务是并行执行的


1 GlobalScope.launch {}
- 官方不建议使用，生命周期和 app 一致，且过程中无法过早取消
- 默认采用 Dispatchers.Defalut

2 CoroutineScope(Dispatchers.IO).launch {}
- 一般需要在 onDestroy 中调用 coroutineScope.cancel 方法手动取消

```
val coroutineScope = CoroutineScope(Dispatchers.IO)
coroutineScope.launch {
    getFileData(fileUrl)
}
```

3 MainScope().launch {}
- 一般需要在 onDestroy 中调用 mainScope.cancel 方法手动取消
- 默认已硬编码为 Dispatchers.Main
- 常见场景是可以配合 by 使用处理


```kotlin
//类委托
class CancelJobDialog() : DialogFragment(), CoroutineScope by MainScope() {

  override fun onDismiss(dialog: DialogInterface) {
        //手动取消
        this.cancel()
        super.onDismiss(dialog)
    }
    
}
```

4 coroutineScope.launch {}
- coroutineScope 是 Lifecycle 的扩展属性
- 默认已硬编码为 Dispatchers.Main.immediate

5 lifecycleScope.launch {}
- - lifecycleScope 就是 lifecycle.coroutineScope
- 默认已硬编码为 Dispatchers.Main.immediate

6 viewModelScope.launch {}
- 默认已硬编码为 Dispatchers.Main.immediate

## async
- 是一个 CoroutineScope 接口的扩展方法，用于创建启动一个协程，不阻塞当前协程，并返回一个 Deferred，内部有一个 await 的挂起函数， Deferred 也实现了 Job 接口，可以看成是一个带返回值的 Job，所以也可以调用 join 方法
- 第一个参数是 CoroutineContext 协程上下文
- 任务是并行执行的
- 可返回结果，通过 await 挂起函数获取返回结果，就是 async return 写的返回值，如果不使用 await，那么就相当于 launch
- 调用 await 不阻塞线程，就是在不阻塞线程的情况下等待该值的完成并继续执行
- 最常用的场景就是同时开启多个请求，等所有的请求结果都拿到后再接着进行后续别的操作

## runBlocking 
- runBlocking 是一个普通的方法，用于创建启动一个协程，并阻塞当前线程，直到协程执行完毕，如果内部有子协程会等子协程执行完成，在 Android 代码中并不常用（不建议在项目中使用），主要用于连接协程和线程
- 第一个参数是 CoroutineContext 协程上下文
- 可返回结果
- 是阻塞线程的
- 可全局创建

## withContext 
- 是一个 suspend 挂起函数，不会创建启动协程，只可能会导致线程的切换，用于在不同的协程上下文中执行代码
- 第一个参数是 CoroutineContext 协程上下文
- 可返回结果，就是 return 写的返回值
- 任务是串行执行的
- 一般就是用来来确保使用的这个函数都是主线程安全的，这意味着调用方就不需要顾虑考虑应该使用哪个线程来执行函数了，可以直接从主线程调用这个函数，可以将 withContext 应用于非常小的函数

## coroutineScope
- 是一个 suspend 挂起函数，不会创建启动协程，用于创建一个新的作用域，并在该作用域内执行指定的代码块
- 它会等待所有子协程完成后才会继续执行后续代码，主要是用于限制子协程的生命周期与父协程相同
- 如果一个子协程发生异常，那么所有其他子协程将被取消，异常会向上传递到父协程

## supervisorScope
- 是一个 suspend 挂起函数，不会创建启动协程，用于创建一个新的作用域，并在该作用域内执行指定的代码块
- 子协程之间是相互独立的。如果一个子协程发生异常，其他子协程不会受到影响，异常需要在子协程内部处理


## launch 和 async
- 通常使用 launch 从常规函数启动新协程，因为常规函数无法调用 await，只有在另一个协程内或在挂起函数内且在执行并行分解时才使用 async
- launch 和 async 处理异常的方式不同。由于 async 希望在某一时刻对 await 进行最终调用，因此它持有异常并将其作为 await 调用的一部分重新抛出。这意味着使用 async 从常规函数启动新协程，则能以静默方式丢弃异常。这些丢弃的异常不会出现在崩溃指标中，也不会在 logcat 中


```kotlin
suspend fun fetchTwoDocs() =
    coroutineScope {
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是并行的
        val deferred1 = async(Dispatchers.IO) { suspendFun_fetchDoc1() }
        val deferred2 = async(Dispatchers.IO) { suspendFun_fetchDoc2() }
        val result1 = deferred1.await()
        val result2 = deferred2.await()
        println("fetchTwoDocs result1=$result1 result2=$result2" )
    }
```

```kotlin
suspend fun fetchTwoDocs() =
    coroutineScope {
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 两个是串行的
        val result1 = withContext(Dispatchers.IO) { suspendFun_fetchDoc1() }
        val result2 = withContext(Dispatchers.IO) { suspendFun_fetchDoc2() }
        println("fetchTwoDocs result1=$result1 result2=$result2" )
    }
```


```kotlin
suspend fun fetchTwoDocs() =
    coroutineScope {
        //如果 async 的 await 这么写就和 withContext 的效果差不多
        //suspendFun_fetchDoc1 和 suspendFun_fetchDoc2 
        val result1 = async(Dispatchers.IO) { suspendFun_fetchDoc1() }.await()
        val result2 = async(Dispatchers.IO) { suspendFun_fetchDoc2() }.await()
        println("fetchTwoDocs result1=$result1 result2=$result2" )
    }
```


```kotlin
//awaitAll
suspend fun fetchTwoDocs() =        // called on any Dispatcher (any thread, possibly也许 Main)
    coroutineScope {
        val deferreds = listOf(     // fetch two docs at the same time
            async(Dispatchers.IO) { fetchDoc(1) },  // async returns a result for the first doc
            async(Dispatchers.IO) { fetchDoc(2) }   // async returns a result for the second doc
        )
        deferreds.awaitAll()        // use awaitAll to wait for both network requests
    }
```


