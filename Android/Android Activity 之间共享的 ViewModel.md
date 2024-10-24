# Android Activity 之间共享的 ViewModel
- 提供一个 Application 作用域的 ViewModel 去实现
- 要尽量避免被滥用
- 按需考虑加数据销毁、资源释放的逻辑

## AppSharedViewModel
```kotlin
class AppSharedViewModel: ViewModel() {
    var testLiveData = MutableLiveData(0)
}
```


```kotlin
class AppApplication : Application(), ViewModelStoreOwner {
    companion object {
        private lateinit var sInstance: AppApplication
        fun getInstance() = sInstance
    }

    override fun onCreate() {
        super.onCreate()
        sInstance = this
    }

    private val appSharedViewModelStore by lazy {
        ViewModelStore()
    }

    override fun getViewModelStore(): ViewModelStore {
        return appSharedViewModelStore
    }
}
```

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        //使用
        val appSharedViewModel = ViewModelProvider(AppApplication.getInstance())[AppSharedViewModel::class.java]
    }
```


## 让 AppSharedViewModel 继承自 AndroidViewModel

```kotlin
class AppSharedViewModel(application: Application) : AndroidViewModel(application) {
    var testLiveData = MutableLiveData(0)
}
```

- 方案1

改写 ViewModel 创建获取的地方传入 AndroidViewModelFactory 实例
```kotlin
val appSharedViewModel = ViewModelProvider(
            AppApplication.getInstance(),
            ViewModelProvider.AndroidViewModelFactory.getInstance(AppApplication.getInstance())
        )[AppSharedViewModel::class.java]
```

- 方案2

改写 Application 实现 HasDefaultViewModelProviderFactory 接口，因为 ViewModelProvider 构造方法里有调用 ViewModelProvider.AndroidViewModelFactory.defaultFactory 方法传入 ViewModelStoreOwner 去判断处理 HasDefaultViewModelProviderFactory 接口的逻辑

```kotlin
class AppApplication : Application(), ViewModelStoreOwner, HasDefaultViewModelProviderFactory {
    companion object {
        private lateinit var sInstance: AppApplication
        fun getInstance() = sInstance
    }

    override fun onCreate() {
        super.onCreate()
        sInstance = this
    }

    private val appSharedViewModelStore by lazy {
        ViewModelStore()
    }
    
    override fun getViewModelStore(): ViewModelStore {
        return appSharedViewModelStore
    }

    override fun getDefaultViewModelProviderFactory(): ViewModelProvider.Factory {
        return ViewModelProvider.AndroidViewModelFactory.getInstance(this)
    }
}
```