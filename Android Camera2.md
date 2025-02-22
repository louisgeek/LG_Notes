# Android Camera2
- API 21 新增
- Camera2 基于 Pipeline 管道、使用时逻辑看起来比较复杂
- Camera2 不是必须先开启预览才能拍照或者录像的
- Camera2 支持一次拍摄多张格式和尺寸都不同的图片
- Camera2 支持连拍

组件包括如下：
- CameraManager 相机管理器，可以获取可用的相机列表以及去打开指定的相机
- CameraDevice 相机设备（类似以前的 Camera），可以用于创建 CameraCaptureSession 和 CaptureRequest
- CameraCaptureSession 相机捕捉会话，可以配合 CaptureRequest 去实现预览
- CaptureRequest 捕获请求，可以设置 Surface（来自 TextureView 或 SurfaceView）
- ImageReader 可以用来获取预览帧数据，是使用 ImageReader 提供的 Surface，再通过监听获取图像信息

## 使用步骤
1. 通过获取 CAMERA_SERVICE 系统服务获取到 CameraManager
2. 利用 CameraManager#getCameraIdList 获取设备可用的 cameraId，利用 CameraManager#getCameraCharacteristics 传入指定相机的 cameraId 获取到这个相机的 CameraCharacteristics 相机特征（类似以前的 Camera 的 CamerInfo 和 Parameters）
3. 通过 CameraManager#openCamera 方法打开相机，并设置 CameraDevice.StateCallback 回调，回调成功后就获得了打开成功的 CameraDevice 相机设备
4. 当 CameraDevice 相机打开成功后，就可以通过 CameraDevice#createCaptureSession 传入 surfaceList 集合和 CameraCaptureSession.StateCallback 回调来创建 CameraCaptureSession 相机捕捉会话
5. 回调成功后就可以利用 CameraCaptureSession#setRepeatingRequest 传入 CameraDevice#createCaptureRequest 创建的带 CameraDevice.TEMPLATE_PREVIEW 类型、预览用的 Surface、自动对焦、曝光、白平衡、闪光灯等配置参数的 CaptureRequest 捕获请求去实现预览画面
6. 然后 Camera2 新增了 ImageReader ，可以用它去获取预览帧数据，可以通过 ImageReader.newInstance 创建出 ImageReader 并通过 setOnImageAvailableListener 设置监听回调，通过监听 onImageAvailable 回调利用 ImageReader#acquireNextImage 去拿到可用的 Image ，然后通过 Image#getPlanes 获取的预览内容的 Plane 集合，取第一个 Plane 然后得到 buffer，这个 buffer 可以转成 bytes，然后可以保存到本地或者转成 bitmap 做展示
 

## createCaptureRequest(templateType)
- TEMPLATE_PREVIEW : 创建预览的请求
- TEMPLATE_STILL_CAPTURE: 创建一个适合于静态图像捕获的请求，图像质量优先于帧速率
- TEMPLATE_RECORD : 创建视频录制的请求
- TEMPLATE_VIDEO_SNAPSHOT : 创建视视频录制时截屏的请求
- TEMPLATE_ZERO_SHUTTER_LAG : 创建一个适用于零快门延迟的请求。在不影响预览帧率的情况下最大化图像质量
- TEMPLATE_MANUAL : 创建一个基本捕获请求，这种请求中所有的自动控制都是禁用的(自动曝光，自动白平衡、自动焦点)


## ImageReader#acquireLatestImage 和 ImageReader#acquireNextImage
- acquireLatestImage 从 ImageReader 队列里获取最新的一帧 Image
- acquireNextImage 从 ImageReader 队列里获取下一帧 Image
- 官方更推荐使用 acquireLatestImage
- acquireLatestImage 里调用了 acquireNextImage，而且最后还会执行 image.close 做释放
- acquireNextImage 更适合在批处理或者后台程序中使用，不恰当的使用本方法将会导致图像出现延迟不断增加，然后是完全失速，看起来像没有新的图像出现
