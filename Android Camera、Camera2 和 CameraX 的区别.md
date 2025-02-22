
# Android Camera、Camera2 和 CameraX 的区别
- Camera 顺序调用、逻辑看起来简单、但已经被标记过时了
- Camera2 通过获取 CAMERA_SERVICE 服务获取到 CameraManagr，并通过 CameraManager.openCamera 方法打开 camera，并注册 camera 开启监听回调，新增了 ImageReader 设置，并通过 ImageReader 设置监听，通过监听回调拿到 ImageReader 获取的预览 buffer，拿出 buffer 之后操作差不多
- Camera2 基于管道、逻辑看起来复杂
- CameraX 是 Jetpack 提供的库，是对 Camera2 的封装，增加了 lifecycle 能力


## Camera
默认出来的数据就是 NV21

## Camera2
出来的是 Image 对象，可以先设置类型然后通过 getPlanes 方法得到 JPEG 或者 YUV_420_888

## CameraX

出来的是 ImageProxy 对象，通过 getPlanes 方法得到 YUV_420_888 的数据 -> NV21 ->通过 YuvImage 转换成 jpeg bytes -> bitmap