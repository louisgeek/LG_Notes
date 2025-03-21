
# Kotlin 协程实践


## 最佳实践
1 通常 suspend 和 withContext 搭配使用，保证调用方不特别关注处理也是主线程安全的，而一般这些逻辑都写在 Repository 的方法里
2 withContext 传入的 CoroutineDispatcher 调度器最好不要硬编码，而是通过 Repository 传入
3 通常 ViewModel 就需要创建协程了，而不是再把 Repository 的 suspend 方法再一层 suspend 暴露给上层视图，视图不应直接触发任何协程来执行业务逻辑，而应将这项工作委托给 ViewModel
4 比如 ViewModel 不要公开可变类型
5 Repository 另一种选择就是方法返回  Flow，就是 Repository 一般存放（suspend functions or Flows）
6 避免使用 GlobalScope

### UseCase
- coroutineScope 块
- supervisorScope
- ensureActive

### Repository
Repository 可以考虑传入 CoroutineScope ，然后配合 .join()

### 协程最好设为可取消
- 协程取消属于协作操作，也就是说，在协程的 Job 被取消后，相应协程在挂起或检查是否存在取消操作之前不会被取消
- 自行处理
```
someScope.launch {
    for(file in files) {
        ensureActive() // Check for cancellation
        readFile(file)
    }
}
```






## 协程内部的线程池


 

## LifecycleOwner 
```kotlin
 fun test01(){
  lifecycle.coroutineScope.launch {

  }
  lifecycleScope.launch {

  }

 }
```




 

## ViewModel 
```kotlin
 fun getDeviceInfo(){
    viewModelScope.launch(Dispatchers.IO) {
          //repository.getDeviceInfo(deviceId)
    }
 }

```
## Repository

```kotlin
//写法1
suspend fun getDeviceInfo(deviceId: String) = withContext(Dispatchers.IO) {
        
}
```

 ```kotlin 
 //写法2  
 suspend fun getDeviceInfo(deviceId: String) {
    withContext(Dispatchers.IO) {

    }
}

```

## LiveData

```kotlin
//写法1
fun getDeviceInfo(deviceId: String) = liveData {
     viewModelScope.launch(Dispatchers.Default) {
       emit("testName")
 }
```

```kotlin
//写法2
 fun getDeviceInfo(userName: String): LiveData<String> {
        return liveData {
            viewModelScope.launch(Dispatchers.Default) {
                emit("testName")
            }
        }
    }
```




## repeatOnLifecycle

