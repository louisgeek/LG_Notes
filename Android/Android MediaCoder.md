
##
给 MediaCodeC 配置 Mime 为 video/avc，H.264数据


MediaCodeC配置
codec.configure


## 同步和异步
- 官方推荐异步




## 同步流程



 
```kotlin
//创建 MediaCodec
- MediaCodec.createEncoderByType

//configure 配置
- mediaCodec.configure

//start 启动 MediaCodec
- mediaCodec.start

//==不断获取待处理的数据流

    //从 MediaCodec 中拿到一个输入缓存区队列的索引 index
    mediaCodec.dequeueInputBuffer
    //传入大于等于 0  的 index 索引获取到输入缓存区的 inputBuffer 数据
    mediaCodec.getInputBuffer
    //通过 put 方法将需要处理的流数据填充进 inputBuffer 里
    inputBuffer.put
    //将 inputBuffer 入队后等待 MediaCodec 处理
    mediaCodec.queueInputBuffer


    //从 MediaCodec 中拿到一个输出缓存区队列的索引 index
    mediaCodec.dequeueOutputBuffer
    //通过传入大于等于 0  的 index 索引获取到输出缓存区的 outputBuffer 数据
    mediaCodec.getOutputBuffer
    //对 outputBuffer 做业务处理
    //比如 outputBuffer 转成 outputBytes 做处理
    //业务处理完成后通过之前这个 index 索引释放
    mediaCodec.releaseOutputBuffer



- stop
- release
```
 





不同安卓设d备de MediaCode c解码得到的 YUV格式不尽相同



getPixelStride() 获取行内连续两个颜色值之间的距离(步长)。
getRowStride() 获取行间像素之间的距离。

COLOR_FormatYUV420Flexible YUV420格式的统称


Android PAI 对 YUV420_888的介绍 (统称 通用)
它是YCbCr的泛化格式，能够表示任何4:2:0的平面和半平面格式，每个分量用8 bits 表示。带有这种格式的图像使用3个独立的Buffer表示，每一个Buffer表示一个颜色平面(Plane)，除了Buffer外，它还提供rowStride、pixelStride来描述对应的Plane。
使用Image的getPlanes()获取plane数组:
Image.Plane[] planes = image.getPlanes();
它保证planes[0] 总是Y ，planes[1] 总是U(Cb)， planes[2]总是V(Cr)。并保证Y-Plane永远不会和U/V交叉（yPlane.getPixelStride()总是返回 1 ）。U/V-Plane总是有相同的rowStride和 pixelStride()（即有：uPlane.getRowStride() == vPlane.getRowStride() 和 uPlane.getPixelStride() == vPlane.getPixelStride();）。