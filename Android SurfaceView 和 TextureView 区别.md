# Android SurfaceView 和 TextureView 区别
- SurfaceView 和 TextureView 是用于绘制复杂图形、视频播放或展示游戏画面的视图组件

## SurfaceView
- Surface 表面，SurfaceView 继承自 View，且有自己的 Surface，会在 WMS 中单独创建 WindowState 窗口，不能进行平移、旋转、缩放等操作，不支持截图
- 普通的 View 需要在 UI 主线程中进行修改更新，而 SurfaceView 是在一个独立的后台子线程中进行绘制和刷新的，不会影响 UI 主线程，也能保证刷新速度
- 使用了双缓冲机制，可以使得展示画面更流畅

### 双缓冲机制
- 一个用于绘制，另一个用于显示
- SurfaceView 在更新视图时用到了两张 Canvas 画布，一个 frontCanvas 和 backCanvas
- frontCanvas：实际显示的canvas。
- backCanvas：存储的是上一次更改前的canvas。

当使用 SurfaceHolder#lockCanvas 获取 Canvas 画布时得到的实际上是 backCanvas 而不是正在显示的 frontCanvas，在获取到的 backCanvas 上画完新的内容（比如 canvas.drawBitmap），再通过 SurfaceHolder#unlockCanvasAndPost 新视图，那么传入的这张 canvas 将替换原来的 frontCanvas 作为新的 frontCanvas 在前台显示，而原来的 frontCanvas 将切换到后台成为 backCanvas


## TextureView 
- Texture 纹理，TextureView 继承自 View，它会将其内容渲染到一个硬件纹理上，这个纹理可以与其他视图层次结构一起显示
- 不会在 WMS 中单独创建窗口，所以它可以和其它普通 View 一样进行移动，旋转，缩放，动画等操作，支持截图


## SurfaceView 和 TextureView 区别
- TextureView 内部有缓冲队列，所以内存占用比 SurfaceView 高
- TextureView 耗电比 SurfaceView 高
- TextureView 默认支持硬件加速，而 SurfaceView 默认不支持硬件加速，但可以通过设置 android:hardwareAccelerated="true" 属性来启用
- SurfaceView 绘制比 TextureView 及时，TextureView 可能会有若干帧的延迟
- SurfaceView 不能进行平移、旋转、缩放等操作，不支持截图，但是 TextureView 支持
- SurfaceView 不支持透明度，而 TextureView 支持透明度
- SurfaceView 可能会出现与其他视图之间层叠的顺序问题，而 TextureView 可以很好地与其他视图元素一起使用