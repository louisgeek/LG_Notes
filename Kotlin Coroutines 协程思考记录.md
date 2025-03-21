



```kotlin
        Log.e(TAG, "testCoroutines:1 currentThreadName=${Thread.currentThread().name}")
        Handler(Looper.getMainLooper()).post {
            Log.e(TAG, "testCoroutines:2 currentThreadName=${Thread.currentThread().name}")
            Thread.sleep(3000)
            Log.e(TAG, "testCoroutines:3 currentThreadName=${Thread.currentThread().name}")
        }
        Log.e(TAG, "testCoroutines:4 currentThreadName=${Thread.currentThread().name}")
```

日志打印

```
 testCoroutines:1 currentThreadName=main
 testCoroutines:4 currentThreadName=main
 testCoroutines:2 currentThreadName=main
 //3秒后打印
 testCoroutines:3 currentThreadName=main
```


```kotlin
        Log.e(TAG, "testCoroutines:1 currentThreadName=${Thread.currentThread().name}")
        CoroutineScope(Dispatchers.Main).launch {
            Log.e(TAG, "testCoroutines:2 currentThreadName=${Thread.currentThread().name}")
            delay(3000) //suspend fun
            Log.e(TAG, "testCoroutines:3 currentThreadName=${Thread.currentThread().name}")
        }
        Log.e(TAG, "testCoroutines:4 currentThreadName=${Thread.currentThread().name}")
 ```

 日志打印

```
 testCoroutines:1 currentThreadName=main
 testCoroutines:4 currentThreadName=main
 testCoroutines:2 currentThreadName=main
 //3秒后打印
 testCoroutines:3 currentThreadName=main
```

