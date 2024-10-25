# Android SurfaceView
- 普通的 View 需要在 UI 主线程中进行修改更新，而 SurfaceView 是在一个独立的后台子线程中进行绘制和刷新
- 对于需要频繁刷新页面的场景，如果普通的 View 的 onDraw 方法中执行的操作比较耗时的话，那么可能会导致主线程阻塞，用户事件的响应速度就受影响了，此时可以改用 SurfaceView，SurfaceView 在子线程中绘制 UI 且使用了双缓冲机制，可以使得展示画面更流畅

## 使用步骤
1. 在布局 xml 文件中添加 SurfaceView，然后在代码中获取到 SurfaceView 实例或者直接在代码里 new 出一个 SurfaceView 实例
2. 通过 SurfaceView#getHolder 方法获取到 SurfaceHolder 对象，然后通过 SurfaceHolder 实例的 addCallback 设置一个回调，包括 surfaceCreated、surfaceChanged 和 surfaceDestroyed 三个方法，Surface 在创建或销毁时会收到对应的回调
3. 一般在 surfaceCreated 方法中启动绘图的线程，一般在 surfaceDestroyed 方法中停止绘图线程
4. 一般在 surfaceCreated 方法中 Camera#open 打开相机，通过 Camera#setPreviewDisplay 传入回调里的 surfaceHolder 参数，然后通过 Camera#startPreview 打开预览，，一般在 surfaceDestroyed 方法中 Camera#stopPreview 停止预览，然后 Camera#release 释放相机


```java
try {
    //获得 canvas 对象
    mCanvas = mSurfaceHolder.lockCanvas();
    if (mCanvas != null) {
       //绘制操作
       //绘制背景
       mCanvas.drawColor(Color.WHITE);
       //绘制 bitmap
       mCanvas.drawBitmap(bitmap, null, rect, null);
    }
} finally {
 if (mCanvas != null) {
     //释放 canvas 对象并提交
     mSurfaceHolder.unlockCanvasAndPost(mCanvas);
   }
}
```

## 生命周期
