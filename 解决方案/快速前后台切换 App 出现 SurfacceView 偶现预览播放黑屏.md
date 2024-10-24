
# 快速前后台切换 App 出现 SurfacceView 偶现预览播放黑屏

问题：
快速前后台切换 App 偶现预览播放黑屏

背景：
1. 生命周期监听 lifecycle.addObserver(defaultLifecycleObserver)
2. 快速前后台切换 App 会出现不走 onStart 和 onStop 的情况
3.  onStart 执行
```kotlin
surfaceView.holder.addCallback(surfaceHolderCallback)
surfaceView.isVisible = true
```
4.  onStop 执行
```kotlin
 surfaceView.isVisible = false 
 ```
5. 为啥不在 onResume 和 onPause 处理？因为业务需要解决另一个问题
6. surfaceHolderCallback 里的 surfaceCreated 执行 
```kotlin
iViewer?.start()
 ```
7.  surfaceHolderCallback 里 surfaceDestroyed 执行
```kotlin
iViewer?.stop ()
holder.removeCallback(this)
 ```

原因：
1. 因为前后台快速切换 surfaceDestroyed 走了，意味着 removecallback 执行了，可是本来需要执行 addCallback 方法就没机会走了
```kotlin
//走的方法
onPause -> surfaceDestroyed -> onResume
```
2. 换句话说 removeCallback 不能直接写在 surfaceDestroyed 方法里，还是应该写在和执行 addCallback 生命周期对应的生命周期方法里
