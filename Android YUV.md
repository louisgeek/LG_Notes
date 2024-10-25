# YUV
## YUV 采样格式
- YUV 4:4:4
- YUV 4:2:2
- YUV 4:2:0 

## YUV 存储格式
- Packed 打包                        三个分量全部交错存放 
- Planar 平面   Y-U-V 或 Y-V-U       三个分量分开存放
- Semi-Planar 半平面  Y-UV 或 Y-VU    Y 分量单独存放，UV 分量交错存放

## YUV 420
### YUV 420P （YUV 420 Planar）
 - I420（YU12）
 - YV12
### YUV 420SP（YUV 420 Semi-Planar）
 - NV12
 - NV21

## YUV 422
### YUV 422P （YUV 422 Planar）
 - I422（YU16）
 - YV16
### YUV 422SP（YUV 422 Semi-Planar）
 - NV16
 - NV61

## NV 系列
- NV 对应都属于 Semi-Planar 系列
- 数字从小到大的顺序对应 UV 的顺序，NV12 对应 Y-UV，而 NV21 对应 Y-VU，NV16 和 NV61 同理
- NV12 中数字 12 代表一个像素占 12 位，其中 Y 占 8 位， U 占 2 位，V 占 2 位，8+2+2=12（12 位就是 1.5 个字节）
- NV16 中数字 16 代表一个像素占 16 位，其中 Y 占 8 位， U 占 4 位，V 占 4 位，8+4+4=16（16 位就是 2 个字节）

### Bitmap 格式
Bitmap.Config.ALPHA_8:   只存储透明度，不存储色值，一个像素占 8 位（1 个字节）
Bitmap.Config.ARGB_4444: ARGB 各用 4 位存储，一个像素占 16 位（2个字节）
Bitmap.Config.ARGB_8888: ARGB 各用 8 位存储，一个像素占 32 位（4个字节）
Bitmap.Config.RGB_565:   只存储色值，不存储透明度，默认不透明，RGB 分别占 5 位，6 位，5 位，一个像素占 16 位（2个字节）


### 图片大小计算
Bitmap ARGB_8888 每个像素有 ARGB 四个字节：            width * height * 4
YUV444 每个像素有一个 Y，一个 U，一个 V，就是三个字节： width * height * 3
YUV422 每个像素有一个 Y，一半的 U 和一半的 V：          width * height + ( width * height * 0.5 ) + ( width * height * 0.5 ) = width * height * 2
YUV420 每个像素有一个 Y，四分之一的 U 和四分之一的 V：  width * height + ( width * height * 0.25 ) + ( width * height * 0.25 ) = width * height * 1.5

1280 * 720 分辨率的图片
ARGB_8888  1280 * 720 * 4 = 3686400（3.52 MB）
YUV444     1280 * 720 * 3 = 2764800（2.64 MB）
YUV422     1280 * 720 * 2 = 1843200（1.76 MB）
YUV420     1280 * 720 * 1.5 = 1382400（1.32 MB）


## Android 里的 YUV

### Camera 默认使用 NV21

### Camera2 的 Image 类型为 YUV_420_888

### YuvImage
- 只支持 ImageFormat.NV21 与 ImageFormat.YUY2 两个格式
- 可以通过 compressToJpeg 方法将 YUV 数据压缩成 JPEG 数据


### ScriptIntrinsicYuvToRGB 
- 使用 Renderscript 内联函数转换 NV21 到 RGBA 再到 Bitmap


## libyuv

libyuv 缩放、旋转、镜像、裁剪等都是围绕着 I420 进行处理的


## 其他
YUV数据包含Padding

