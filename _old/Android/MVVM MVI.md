`MVVM`的双向数据绑定主要通过`DataBinding`实现



MVVM 不足 槽点

ViewModel 里声明 2 份 LiveData，可变的和不可变的

```kotlin
//ViewModel 里的模板代码
private val _loading: MutableLiveData<String> = MutableLiveData()
val loading: LiveData<String> = _loading
...
```





MVI

更加强调数据的单向流动和唯一数据源



M  Model - UiState

V View

I Intent 请求



### 单向数据流



所有状态都通过一个`LiveData`来管理，也带来了一个严重的问题，即**页面不支持局部刷新**