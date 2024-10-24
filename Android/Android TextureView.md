# Android TextureView
 

## 使用步骤
1. 在布局 xml 文件中添加 TextureView，然后在代码中获取到 TextureView 实例或者直接在代码里 new 出一个 TextureView 实例
2. 通过设置 setSurfaceTextureListener 方法设置一个监听回调，包括 onSurfaceTextureAvailable、onSurfaceTextureSizeChanged、onSurfaceTextureDestroyed 和 onSurfaceTextureUpdated 四个方法
3. 一般在 onSurfaceTextureAvailable 方法中启动绘图的线程，一般在 onSurfaceTextureDestroyed 方法中停止绘图线程和回调里的 surfaceTexture 参数实例的 release 释放操作
4. 一般在 onSurfaceTextureAvailable 方法中 Camera#open 打开相机，通过 Camera#setPreviewTexture 传入回调里的 surfaceTexture 参数，然后通过 Camera#startPreview 打开预览，，一般在 onSurfaceTextureDestroyed 方法中 Camera#stopPreview 停止预览，然后 Camera#release 释放相机



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
