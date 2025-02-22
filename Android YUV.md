# Android YUV
- YUV 是一种颜色编码（颜色模型、色彩空间）方法（也称为 YCbCr，其中的两个 C 就是色度这个单词的缩写），将图像的颜色信息分解为三个分量，Y 表示亮度（明亮度，Luminance 亮度或 Luma 明度），也就是灰度值（灰阶值），而 U 和 V 表示色度（色彩及饱和度，Chrominance 色度或 Chroma 浓度），U 代表 Blue 蓝色色度分量（即 Cb），V 代表 Red 红色色度分量（即 Cr），考虑到人眼对亮度信息比对色度信息更加敏感，因此通过分离亮度和色度，并对色度进行适当的压缩（降采样），可以在保证较高视觉效果的前提下，大大减少数据量，因此被广泛应用于电视信号传输、摄像监控系统、图像和视频处理等领域中，特别是在视频压缩、传输和存储中起着很关键作用
- 另一种常见的颜色编码格式就是 RGB，每个像素使用三个颜色通道来表示，假设一张 1280 * 720 分辨率的标准图片，占用存储空间 1280 * 720 * 3 = 2764800 Byte（2.64 MB），而由于 YUV 可以针对色度进行压缩（降采样），所以 YUV 的数据量通常会比 RGB 要小，更适合视频的传输和存储
- YUV 相当于对原始的 RGB 信息进行重编码，压缩减少一些色彩信息，在从黑白电视向彩色电视的过渡期，电视系统可以通过 YUV 的特性提供对黑白电视的兼容性，同时电视广播系统可以通过 YUV 的来更加节省带宽
- 

！！！1  而JPEG这个常见的图片压缩编码也使用了YUV作为处理和存储图像时使用的色彩模型，
常用于视频和图像处理领域 广泛应用于视频处理和图像编码领域
在YUV颜色空间中，每个颜色都有一个亮度信号Y和两个色度信号U和V。
这种颜色空间的特别之处在于亮度信号和色度信号的分离，使得在不影响颜色的情况下可以改变图像的亮度。

## YUV 采样格式
- YUV 4:4:4 每个 Y 分量都对应一个 U 分量和一个 V 分量，意味着色度信息没有被压缩，色彩还原最准确，每个像素都包含完整的 Y、U、V 三个分量，占用的大小和 RGB 方式是一样的
- YUV 4:2:2 每两个 Y 分量共用一组 U、V 分量，第一和第二个像素点共用一组 U、V 分量，第三和第四个像素点共用了一组 U、V 分量（Y:U:V 就是 2:1:1）
- YUV 4:2:0 每四个 Y（上下两行相邻的四个像素点）共用一组 U、V 分量，在第一行采集 Y 和其中一种色度分量（比如 U 分量），在第二行采集 Y 和其中的另一个色度分量（比如 V 分量）
- 所以在上下两行相邻的四个像素点的情况下，YUV 4:4:4 对应的 Y-U-V 的数量就是 4-4-4，YUV 4:2:2 对应的 Y-U-V 的数量就是 4-2-2，YUV 4:2:0 对应的 Y-U-V 的数量就是 4-1-1

## YUV 存储格式
- Packed 打包（也叫 Interleaved 交错） 三个分量全部交错存放，就是按照 YUV YUV YUV YUV ... 这样存放
- Planar 平面 Y-U-V 或 Y-V-U          三个分量分开存放
- Semi-Planar 半平面 Y-UV 或 Y-VU     Y 分量单独存放，UV 分量交错存放

```java
//举例：YUV 420 每个像素总共占 12 位，其中 Y 占 8 位（1 个字节），U 占 2 位，V 占 2 位
//Planar 平面 Y-U-V
YYYYYYYY UU VV 就是 I420(YU12)

//Planar 平面 Y-V-U 
YYYYYYYY VV UU 就是 YV12

//Semi-Planar 半平面 Y-UV
YYYYYYYY UVUV  就是 NV12

//Semi-Planar 半平面 Y-VU
YYYYYYYY VUVU  就是 NV21
```

## YUV 420
YUV 420P （YUV 420 Planar）
 - I420（YU12）
 - YV12

YUV 420SP（YUV 420 Semi-Planar）
 - NV12
 - NV21

## YUV 422
YUV 422P （YUV 422 Planar）
- I422（YU16）
- YV16

YUV 422SP（YUV 422 Semi-Planar）
- NV16
- NV61

YUV 422 Interleaved（YUV 422 Packed）
- YUY2（YUYV）
- YVYU


## NV 系列
- NV 系列对应都属于 Semi-Planar 系列
- 数字从小到大的顺序对应 UV 的顺序，NV12 对应 Y-UV，而 NV21 对应 Y-VU，NV16 和 NV61 同理
- NV12 中数字 12 代表一个像素占 12 位，其中 Y 占 8 位，U 占 2 位，V 占 2 位，8+2+2=12（12 位就是 1.5 个字节）
- NV16 中数字 16 代表一个像素占 16 位，其中 Y 占 8 位，U 占 4 位，V 占 4 位，8+4+4=16（16 位就是 2 个字节）

！！！  
RGB24 RGB 三个分量各占 8 位
RGB32 RGB 三个分量各占 8 位（剩下的 8 位不用）
ARGB32 ARGB 四个分量各占 8 位

## Bitmap 格式
Bitmap.Config.ALPHA_8: 只存储透明度，不存储色值，一个像素占 8 位（1 个字节）
Bitmap.Config.ARGB_4444: ARGB 各用 4 位存储，一个像素占 16 位（2个字节）
Bitmap.Config.ARGB_8888: ARGB 各用 8 位存储，一个像素占 32 位（4个字节）
Bitmap.Config.RGB_565: 只存储色值，不存储透明度，默认不透明，RGB 分别占 5 位，6 位，5 位，一个像素占 16 位（2个字节）


### 图片大小计算
Bitmap ARGB_8888 每个像素有 ARGB 四个字节：            width * height * 4
YUV444 每个像素有一个 Y，一个 U，一个 V，就是三个字节： width * height * 3
YUV422 每个像素有一个 Y，一半的 U 和一半的 V：          width * height + ( width * height * 0.5 ) + ( width * height * 0.5 ) = width * height * 2
YUV420 每个像素有一个 Y，四分之一的 U 和四分之一的 V：  width * height + ( width * height * 0.25 ) + ( width * height * 0.25 ) = width * height * 1.5

1280 * 720 分辨率的图片
ARGB_8888  1280 * 720 * 4 = 3686400 Byte（3.52 MB）
YUV444     1280 * 720 * 3 = 2764800 Byte（2.64 MB）
YUV422     1280 * 720 * 2 = 1843200 Byte（1.76 MB）
YUV420     1280 * 720 * 1.5 = 1382400（1.32 MB）

## Android 里的 YUV
- Camera 默认的 PreviewFormat 预览格式是 ImageFormat.NV21
- Camera2 的 Image 类型为 ImageFormat.YUV_420_888

## YuvImage
- 只支持 ImageFormat.NV21 与 ImageFormat.YUY2 两个格式
- 可以通过 compressToJpeg 方法将 YUV 数据压缩成 JPEG 数据


## ScriptIntrinsicYuvToRGB 
- 使用 Renderscript 内联函数转换 NV21 到 RGBA 再到 Bitmap


## libyuv
- libyuv 缩放、旋转、镜像、裁剪等都是围绕着 I420 进行处理的


## 总结
- YUV 的原理就是把亮度和色度分离，人眼对亮度的敏感超过色度，把色度信息减少一点，所以 YUV 数据能够比 RGB 小
- 单独只有 Y 数据就可以形成一张图片，只不过这张图片是灰色的，所以彩色 YUV 转黑白 YUV 转换非常容易，这一特性被应用在电视信号传输上
- 
- YUV数据包含Padding

