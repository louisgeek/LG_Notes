android.graphics.ImageFormat  

> API 30



1.无论android还是ios都不能直接从摄像头取出颜色空间为i420的数据，所以在编码前需要进行格式转换。

2.而且由于所取图像得分辨率必须是摄像头所提供分辨率中得一组，所以有可能需要裁剪。

3.另外由于1)想让无论用户哪个方向拿手机所录的视频内容永远“头朝上”，

  2)摄像头默认返回图像为横屏图像（宽大于长）所以需要旋转。

4.前置摄像头需要镜像。

IOS 好像只有 NV12

ffmpeg sws filter 或者opencv cvtcolor 可以实现类似操作，但据说 libyuv 效率最好



YUV是一种[颜色](https://baike.baidu.com/item/颜色)[编码](https://baike.baidu.com/item/编码)方法

用来表达颜色的系统

RGB 色彩模式是工业界的一种颜色标准，是通过对红(R)、绿(G)、蓝(B)三个颜色通道的变化以及它们相互之间的叠加来得到各式各样的颜色的，RGB即是代表红、绿、蓝三个通道的颜色，这个标准几乎包括了人类视力所能感知的所有颜色，是目前运用最广的颜色系统之一。

YUV 

YUV的原理是把亮度与色度分离，研究证明,人眼对亮度的敏感超过色度。利用这个原理，可以把色度信息减少一点，人眼也无法查觉这一点。YUV三个字母中，其中”Y”表示明亮度（Lumina nce或Luma），也就是灰阶值；而”U”和”V”表示的则是色度（Chrominance或Chroma），作用是描述影像色彩及饱和度，用于指定像素的颜色。用这个三个字母好象就是通道命令重点内容 

单独只有y数据就可以形成一张图片，只不过这张图片是灰色的

使用YUV的优点有两个: 

1,彩色YUV图像转黑白YUV图像转换非常简单，这一特性用在于电视信号上。 

2,YUV是数据总尺寸小于RGB格式

色度（Chrominance/color）和浓度（Chroma）



## YCbCr 颜色模式（YUV）

我们常说的 YUV ，其实指的是 YCbCr

YUV 原理是把亮度与色度分离

Y 代表灰阶值（明亮度） UV 代表色度（色彩及饱和度）



Y = Y

U = Cb

V = Cr



信号/分量



Y   亮度分量

C = Color  色度分量

b = blue     Cb 蓝色色度分量

r = red     Cr 红色色度分量



yuv发明缘由：（视频信号传输）

> 人眼对色度的敏感程度要低于对亮度的敏感程度

由于人肉眼对 Y 的敏感度远远超过对 U 和 V 的敏感度，有时候可以通过多个 Y 分量共用一组 U 或 V 的方式，这样既可以极大地节省空间，又能达到观感质量不太损失的效果;

把色度信息减少一点，人眼也无法查觉这一点

**眼睛对于亮和暗的分辨要比对颜色的分辨精细一些**。正是 因为这个，在我们的视频存储中，没有必要存储全部颜色信号。既然眼睛看不见，那为什么要浪费存储空间（或者说是金钱）来存储它们

消费用录像带就得益于将录像带上的更多带宽留给黑—白信号（被称作“亮度”），将稍少的带宽留给彩色信号（被称作“色度”）

那么为什么有了RGB我们仍然需要YUV呢？我们要回到人类刚拥有彩色电视的时候，在那段从黑白电视向彩色电视的过渡期，电视系统需要提供对黑白电视的兼容性，另外还要考虑到电视广播系统那有限的带宽，如果使用RGB颜色模型，那么传输带宽就是原来的三倍。主要是以上两个原因，能够兼容黑白电视系统和更为节省带宽的YUV色彩模型就被发明了出来，它与RGB之间是无损转换的。

亮度信息与色彩信息相分离的设计使得YUV可以减少一些色彩信息以达到节省传输带宽和保存体积的目的。因为相较于色彩，人眼对于亮度信息更为敏感，所以可以在色彩信息上面进行取舍来达到节约大小的目的，通过引入采样的方式，YUV对原始的RGB信息进行重编码，目前在视频中最常见的就是YUV420式编码，Y平面的信息完全保留，而UV这两个色度平面的信息交错保留，并且精度只有Y平面的一半，最终图像、视频的体积也就少了很多，而画质损失实际是被控制在一个合理的范围内。



YUV下还有很多不同的具体编码方式，比如视频中常见的NV12、YV12等，而JPEG这个常见的图片压缩编码也使用了YUV作为处理和存储图像时使用的色彩模型，可以说，我们虽然没有直接接触到YUV色彩模型，但是几乎是时时刻刻都在用它。而YPbPr、YCbCr只是YUV在不同领域中的具现化罢了，其实就是一个东西。



YUV采样方式-常见的采样格式：YUV444、YUV422和YUV420。（数字部分表示二次采样率，J:a:b模式，J：表示参考像素值，也可以理解为亮度样本数，a：表示水平相邻色度样本数；b：表示垂直方向相邻色度样本数）

 

每秒传输的帧数，视频帧率不小于24fps，人眼会觉得视频是连贯的，即“视觉暂留”。



## 按照数据大小分类

色度信号分辨率最高的格式是4:4:4  *色度信号分辨率最低的格式4:2:0*  4:1:1 表示4:1的水平取样，垂直完全采样。

### YUV 420

4 个 Y 分量共用一套 UV 分量

四像素中水平采样两个色度样本，垂直不采样，即八个像素中采用两个色度样本，是全采样的四分之一。

### YUV 422

 2 个 Y 分量共用一套 UV 分量

四像素中水平采样两个色度样本，垂直采用两个样本，即八个像素中采用四个色度样本，是全采样的二分之一。



### YUV 444

一个 Y 分量使用一套 UV 分量

每4点Y采样，就有相对应的4点Cb和4点Cr。换句话说，在这种格式中，色度信号的分辨率和亮度信号的分辨率是相同的。

全采样，每个像素的Y、U、V通道都保留；



## 按照排列方式分类

### Planar

YUV 三个分量分开存放

### Semi-Planar

Y 分量单独存放，UV 分量交错存放

### Packed (or Interleaved)

YUV 三个分量全部交错存放





|         | Planar          | Semi-Planar | Packed/Interleaved |      |
| ------- | --------------- | ----------- | ------------------ | ---- |
| YUV 411 |                 |             |                    |      |
| YUV 420 | I420(YU12) YV12 | NV12 NV21   |                    |      |
| YUV 422 | I422(YU16) YV16 | NV16 NV61   | YUYV(YUY2) UYVY    |      |
| YUV 444 | I444 YV24       | NV24 NV42   | AYUV               |      |
|         |                 |             |                    |      |

所以

格式的名称以“ p”结尾的一般都是planar格式,如在ffmpeg 中的yuvj420p格式
格式的名称以“ sp”结尾的一般都是semi-planar格式



YYYYYYYY UU VV -> I420(YU12)

YYYYYYYY VV UU -> YV12

YYYYYYYY UVUV -> NV12

YYYYYYYY VUVU -> NV21









## Y'UV, YUV, [YCbCr](https://baike.baidu.com/item/YCbCr)，[YPbPr](https://baike.baidu.com/item/YPbPr)等专有名词都可以称为YUV

YUV、Y‘UV、YCbCr, YPbPr

YUV 里的 Y 用来代指 luminance 表示的是自然颜色的亮度

Y‘UV 里的 Y‘ 用来代指 luma / brightness表示的是**经过伽马压缩**之后电信号的强度

YUV 一般用来代指 YCbCr,用来表示文件的编码格式，用于数字视频的编码，

YPbPr 颜色模型常常用在模拟分量视频中

模拟 YUV  = YPbPr 

数字 YUV = YCbCr

而YCbCr则是用来描述数字的视频信号，适合视频与图片压缩以及传输，例如MPEG、JPEG

,JPEG、MPEG，H264均采用此格式。







YUV文件播放或编码必须要知道分辨率和YUV具体的格式 **因此在YUV文件命名时建议名字包含YUV格式名称和分辨率信息**

如果使用播放器单独播放U分量和V分量的话，是看不到色彩的，因为手机屏幕和显示器是RGB格式，电视是YUV格式

因为YUV存储的数据要比RGB小很多。

YUV444的话，和RGB888一样都是24bits





RGB 颜色模式 三原⾊

R = Red

G = Green

B = Blue





Y

亮度



U





V 



##  ImageFormat.NV21

YCrCb 

 ImageFormat.NV21 是 android.hardware.Camera  默认的 PreviewFormat 预览格式

- ImageFormat.NV21 所有 Android 设备都支持

- ImageFormat.YV12 从 Android API 12 开始都支持



## ImageFormat.YUV_420_888

YCbCr



```
planar, semiplanar or interleaved
```

| CameraFormat | PixelFormat | PreviewFormat | PictureFormat |
| ------------ | ----------- | ------------- | ------------- |
| yuv422sp     | NV16        |               |               |
| yuv420sp     | NV21        |               |               |
| yuv422i-yuyv | YUY2        |               |               |
| yuv420p      | YV12        |               |               |
| rgb565       | RGB_565     |               |               |
| jpeg         | JPEG        |               |               |
| bayer-rggb   |             |               |               |



彩色电视采用YUV空间正是为了用亮度信号Y解决彩色电视机与黑白电视机的兼容问题，使黑白电视机也能接收彩色电视信号。



RGB_565



​             YV12,
​             Y8,
​             Y16,
​             NV16,
​             NV21,
​             YUY2,



​             JPEG,
​             DEPTH_JPEG,



​             YUV_420_888,
​             YUV_422_888,
​             YUV_444_888,



​             FLEX_RGB_888,
​             FLEX_RGBA_8888,



​             RAW_SENSOR,
​             RAW_PRIVATE,
​             RAW10,
​             RAW12,
​             DEPTH16,
​             DEPTH_POINT_CLOUD,
​             RAW_DEPTH,
​             PRIVATE,
​             HEIC



```java
/*
 * Copyright (C) 2010 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package android.graphics;

import android.annotation.IntDef;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

public class ImageFormat {
     /** @hide */
     @Retention(RetentionPolicy.SOURCE)
     @IntDef(value = {
             UNKNOWN,
             RGB_565,
             YV12,
             Y8,
             Y16,
             NV16,
             NV21,
             YUY2,
             JPEG,
             DEPTH_JPEG,
             YUV_420_888,
             YUV_422_888,
             YUV_444_888,
             FLEX_RGB_888,
             FLEX_RGBA_8888,
             RAW_SENSOR,
             RAW_PRIVATE,
             RAW10,
             RAW12,
             DEPTH16,
             DEPTH_POINT_CLOUD,
             RAW_DEPTH,
             PRIVATE,
             HEIC
     })
     public @interface Format {
     }

    /*
     * these constants are chosen to be binary compatible with their previous
     * location in PixelFormat.java
     */

    public static final int UNKNOWN = 0;

    /**
     * RGB format used for pictures encoded as RGB_565. See
     * {@link android.hardware.Camera.Parameters#setPictureFormat(int)}.
     */
    public static final int RGB_565 = 4;

    /**
     * <p>Android YUV format.</p>
     *
     * <p>This format is exposed to software decoders and applications.</p>
     *
     * <p>YV12 is a 4:2:0 YCrCb planar format comprised of a WxH Y plane followed
     * by (W/2) x (H/2) Cr and Cb planes.</p>
     *
     * <p>This format assumes
     * <ul>
     * <li>an even width</li>
     * <li>an even height</li>
     * <li>a horizontal stride multiple of 16 pixels</li>
     * <li>a vertical stride equal to the height</li>
     * </ul>
     * </p>
     *
     * <pre> y_size = stride * height
     * c_stride = ALIGN(stride/2, 16)
     * c_size = c_stride * height/2
     * size = y_size + c_size * 2
     * cr_offset = y_size
     * cb_offset = y_size + c_size</pre>
     *
     * <p>For the {@link android.hardware.camera2} API, the {@link #YUV_420_888} format is
     * recommended for YUV output instead.</p>
     *
     * <p>For the older camera API, this format is guaranteed to be supported for
     * {@link android.hardware.Camera} preview images since API level 12; for earlier API versions,
     * check {@link android.hardware.Camera.Parameters#getSupportedPreviewFormats()}.
     *
     * <p>Note that for camera preview callback use (see
     * {@link android.hardware.Camera#setPreviewCallback}), the
     * <var>stride</var> value is the smallest possible; that is, it is equal
     * to:
     *
     * <pre>stride = ALIGN(width, 16)</pre>
     *
     * @see android.hardware.Camera.Parameters#setPreviewCallback
     * @see android.hardware.Camera.Parameters#setPreviewFormat
     * @see android.hardware.Camera.Parameters#getSupportedPreviewFormats
     * </p>
     */
    public static final int YV12 = 0x32315659;

    /**
     * <p>Android Y8 format.</p>
     *
     * <p>Y8 is a YUV planar format comprised of a WxH Y plane only, with each pixel
     * being represented by 8 bits. It is equivalent to just the Y plane from {@link #YV12}
     * format.</p>
     *
     * <p>This format assumes
     * <ul>
     * <li>an even width</li>
     * <li>an even height</li>
     * <li>a horizontal stride multiple of 16 pixels</li>
     * </ul>
     * </p>
     *
     * <pre> size = stride * height </pre>
     *
     * <p>For example, the {@link android.media.Image} object can provide data
     * in this format from a {@link android.hardware.camera2.CameraDevice} (if
     * supported) through a {@link android.media.ImageReader} object. The
     * {@link android.media.Image#getPlanes() Image#getPlanes()} will return a
     * single plane containing the pixel data. The pixel stride is always 1 in
     * {@link android.media.Image.Plane#getPixelStride()}, and the
     * {@link android.media.Image.Plane#getRowStride()} describes the vertical
     * neighboring pixel distance (in bytes) between adjacent rows.</p>
     *
     * @see android.media.Image
     * @see android.media.ImageReader
     * @see android.hardware.camera2.CameraDevice
     */
    public static final int Y8 = 0x20203859;

    /**
     * <p>Android Y16 format.</p>
     *
     * Y16 is a YUV planar format comprised of a WxH Y plane, with each pixel
     * being represented by 16 bits. It is just like {@link #Y8}, but has 16
     * bits per pixel (little endian).</p>
     *
     * <p>This format assumes
     * <ul>
     * <li>an even width</li>
     * <li>an even height</li>
     * <li>a horizontal stride multiple of 16 pixels</li>
     * </ul>
     * </p>
     *
     * <pre> y_size = stride * height </pre>
     *
     * <p>For example, the {@link android.media.Image} object can provide data
     * in this format from a {@link android.hardware.camera2.CameraDevice}
     * through a {@link android.media.ImageReader} object if this format is
     * supported by {@link android.hardware.camera2.CameraDevice}.</p>
     *
     * @see android.media.Image
     * @see android.media.ImageReader
     * @see android.hardware.camera2.CameraDevice
     *
     * @hide
     */
    public static final int Y16 = 0x20363159;

    /**
     * YCbCr format, used for video.
     *
     * <p>For the {@link android.hardware.camera2} API, the {@link #YUV_420_888} format is
     * recommended for YUV output instead.</p>
     *
     * <p>Whether this format is supported by the old camera API can be determined by
     * {@link android.hardware.Camera.Parameters#getSupportedPreviewFormats()}.</p>
     *
     */
    public static final int NV16 = 0x10;

    /**
     * YCrCb format used for images, which uses the NV21 encoding format.
     *
     * <p>This is the default format
     * for {@link android.hardware.Camera} preview images, when not otherwise set with
     * {@link android.hardware.Camera.Parameters#setPreviewFormat(int)}.</p>
     *
     * <p>For the {@link android.hardware.camera2} API, the {@link #YUV_420_888} format is
     * recommended for YUV output instead.</p>
     */
    public static final int NV21 = 0x11;

    /**
     * YCbCr format used for images, which uses YUYV (YUY2) encoding format.
     *
     * <p>For the {@link android.hardware.camera2} API, the {@link #YUV_420_888} format is
     * recommended for YUV output instead.</p>
     *
     * <p>This is an alternative format for {@link android.hardware.Camera} preview images. Whether
     * this format is supported by the camera hardware can be determined by
     * {@link android.hardware.Camera.Parameters#getSupportedPreviewFormats()}.</p>
     */
    public static final int YUY2 = 0x14;

    /**
     * Compressed JPEG format.
     *
     * <p>This format is always supported as an output format for the
     * {@link android.hardware.camera2} API, and as a picture format for the older
     * {@link android.hardware.Camera} API</p>
     */
    public static final int JPEG = 0x100;

    /**
     * Depth augmented compressed JPEG format.
     *
     * <p>JPEG compressed main image along with XMP embedded depth metadata
     * following ISO 16684-1:2011(E).</p>
     */
    public static final int DEPTH_JPEG = 0x69656963;

    /**
     * <p>Multi-plane Android YUV 420 format</p>
     *
     * <p>This format is a generic YCbCr format, capable of describing any 4:2:0
     * chroma-subsampled planar or semiplanar buffer (but not fully interleaved),
     * with 8 bits per color sample.</p>
     *
     * <p>Images in this format are always represented by three separate buffers
     * of data, one for each color plane. Additional information always
     * accompanies the buffers, describing the row stride and the pixel stride
     * for each plane.</p>
     *
     * <p>The order of planes in the array returned by
     * {@link android.media.Image#getPlanes() Image#getPlanes()} is guaranteed such that
     * plane #0 is always Y, plane #1 is always U (Cb), and plane #2 is always V (Cr).</p>
     *
     * <p>The Y-plane is guaranteed not to be interleaved with the U/V planes
     * (in particular, pixel stride is always 1 in
     * {@link android.media.Image.Plane#getPixelStride() yPlane.getPixelStride()}).</p>
     *
     * <p>The U/V planes are guaranteed to have the same row stride and pixel stride
     * (in particular,
     * {@link android.media.Image.Plane#getRowStride() uPlane.getRowStride()}
     * == {@link android.media.Image.Plane#getRowStride() vPlane.getRowStride()} and
     * {@link android.media.Image.Plane#getPixelStride() uPlane.getPixelStride()}
     * == {@link android.media.Image.Plane#getPixelStride() vPlane.getPixelStride()};
     * ).</p>
     *
     * <p>For example, the {@link android.media.Image} object can provide data
     * in this format from a {@link android.hardware.camera2.CameraDevice}
     * through a {@link android.media.ImageReader} object.</p>
     *
     * @see android.media.Image
     * @see android.media.ImageReader
     * @see android.hardware.camera2.CameraDevice
     */
    public static final int YUV_420_888 = 0x23;

    /**
     * <p>Multi-plane Android YUV 422 format</p>
     *
     * <p>This format is a generic YCbCr format, capable of describing any 4:2:2
     * chroma-subsampled (planar, semiplanar or interleaved) format,
     * with 8 bits per color sample.</p>
     *
     * <p>Images in this format are always represented by three separate buffers
     * of data, one for each color plane. Additional information always
     * accompanies the buffers, describing the row stride and the pixel stride
     * for each plane.</p>
     *
     * <p>The order of planes in the array returned by
     * {@link android.media.Image#getPlanes() Image#getPlanes()} is guaranteed such that
     * plane #0 is always Y, plane #1 is always U (Cb), and plane #2 is always V (Cr).</p>
     *
     * <p>In contrast to the {@link #YUV_420_888} format, the Y-plane may have a pixel
     * stride greater than 1 in
     * {@link android.media.Image.Plane#getPixelStride() yPlane.getPixelStride()}.</p>
     *
     * <p>The U/V planes are guaranteed to have the same row stride and pixel stride
     * (in particular,
     * {@link android.media.Image.Plane#getRowStride() uPlane.getRowStride()}
     * == {@link android.media.Image.Plane#getRowStride() vPlane.getRowStride()} and
     * {@link android.media.Image.Plane#getPixelStride() uPlane.getPixelStride()}
     * == {@link android.media.Image.Plane#getPixelStride() vPlane.getPixelStride()};
     * ).</p>
     *
     * <p>For example, the {@link android.media.Image} object can provide data
     * in this format from a {@link android.media.MediaCodec}
     * through {@link android.media.MediaCodec#getOutputImage} object.</p>
     *
     * @see android.media.Image
     * @see android.media.MediaCodec
     */
    public static final int YUV_422_888 = 0x27;

    /**
     * <p>Multi-plane Android YUV 444 format</p>
     *
     * <p>This format is a generic YCbCr format, capable of describing any 4:4:4
     * (planar, semiplanar or interleaved) format,
     * with 8 bits per color sample.</p>
     *
     * <p>Images in this format are always represented by three separate buffers
     * of data, one for each color plane. Additional information always
     * accompanies the buffers, describing the row stride and the pixel stride
     * for each plane.</p>
     *
     * <p>The order of planes in the array returned by
     * {@link android.media.Image#getPlanes() Image#getPlanes()} is guaranteed such that
     * plane #0 is always Y, plane #1 is always U (Cb), and plane #2 is always V (Cr).</p>
     *
     * <p>In contrast to the {@link #YUV_420_888} format, the Y-plane may have a pixel
     * stride greater than 1 in
     * {@link android.media.Image.Plane#getPixelStride() yPlane.getPixelStride()}.</p>
     *
     * <p>The U/V planes are guaranteed to have the same row stride and pixel stride
     * (in particular,
     * {@link android.media.Image.Plane#getRowStride() uPlane.getRowStride()}
     * == {@link android.media.Image.Plane#getRowStride() vPlane.getRowStride()} and
     * {@link android.media.Image.Plane#getPixelStride() uPlane.getPixelStride()}
     * == {@link android.media.Image.Plane#getPixelStride() vPlane.getPixelStride()};
     * ).</p>
     *
     * <p>For example, the {@link android.media.Image} object can provide data
     * in this format from a {@link android.media.MediaCodec}
     * through {@link android.media.MediaCodec#getOutputImage} object.</p>
     *
     * @see android.media.Image
     * @see android.media.MediaCodec
     */
    public static final int YUV_444_888 = 0x28;

    /**
     * <p>Multi-plane Android RGB format</p>
     *
     * <p>This format is a generic RGB format, capable of describing most RGB formats,
     * with 8 bits per color sample.</p>
     *
     * <p>Images in this format are always represented by three separate buffers
     * of data, one for each color plane. Additional information always
     * accompanies the buffers, describing the row stride and the pixel stride
     * for each plane.</p>
     *
     * <p>The order of planes in the array returned by
     * {@link android.media.Image#getPlanes() Image#getPlanes()} is guaranteed such that
     * plane #0 is always R (red), plane #1 is always G (green), and plane #2 is always B
     * (blue).</p>
     *
     * <p>All three planes are guaranteed to have the same row strides and pixel strides.</p>
     *
     * <p>For example, the {@link android.media.Image} object can provide data
     * in this format from a {@link android.media.MediaCodec}
     * through {@link android.media.MediaCodec#getOutputImage} object.</p>
     *
     * @see android.media.Image
     * @see android.media.MediaCodec
     */
    public static final int FLEX_RGB_888 = 0x29;

    /**
     * <p>Multi-plane Android RGBA format</p>
     *
     * <p>This format is a generic RGBA format, capable of describing most RGBA formats,
     * with 8 bits per color sample.</p>
     *
     * <p>Images in this format are always represented by four separate buffers
     * of data, one for each color plane. Additional information always
     * accompanies the buffers, describing the row stride and the pixel stride
     * for each plane.</p>
     *
     * <p>The order of planes in the array returned by
     * {@link android.media.Image#getPlanes() Image#getPlanes()} is guaranteed such that
     * plane #0 is always R (red), plane #1 is always G (green), plane #2 is always B (blue),
     * and plane #3 is always A (alpha). This format may represent pre-multiplied or
     * non-premultiplied alpha.</p>
     *
     * <p>All four planes are guaranteed to have the same row strides and pixel strides.</p>
     *
     * <p>For example, the {@link android.media.Image} object can provide data
     * in this format from a {@link android.media.MediaCodec}
     * through {@link android.media.MediaCodec#getOutputImage} object.</p>
     *
     * @see android.media.Image
     * @see android.media.MediaCodec
     */
    public static final int FLEX_RGBA_8888 = 0x2A;

    /**
     * <p>General raw camera sensor image format, usually representing a
     * single-channel Bayer-mosaic image. Each pixel color sample is stored with
     * 16 bits of precision.</p>
     *
     * <p>The layout of the color mosaic, the maximum and minimum encoding
     * values of the raw pixel data, the color space of the image, and all other
     * needed information to interpret a raw sensor image must be queried from
     * the {@link android.hardware.camera2.CameraDevice} which produced the
     * image.</p>
     */
    public static final int RAW_SENSOR = 0x20;

    /**
     * <p>Private raw camera sensor image format, a single channel image with
     * implementation depedent pixel layout.</p>
     *
     * <p>RAW_PRIVATE is a format for unprocessed raw image buffers coming from an
     * image sensor. The actual structure of buffers of this format is
     * implementation-dependent.</p>
     *
     */
    public static final int RAW_PRIVATE = 0x24;

    /**
     * <p>
     * Android 10-bit raw format
     * </p>
     * <p>
     * This is a single-plane, 10-bit per pixel, densely packed (in each row),
     * unprocessed format, usually representing raw Bayer-pattern images coming
     * from an image sensor.
     * </p>
     * <p>
     * In an image buffer with this format, starting from the first pixel of
     * each row, each 4 consecutive pixels are packed into 5 bytes (40 bits).
     * Each one of the first 4 bytes contains the top 8 bits of each pixel, The
     * fifth byte contains the 2 least significant bits of the 4 pixels, the
     * exact layout data for each 4 consecutive pixels is illustrated below
     * ({@code Pi[j]} stands for the jth bit of the ith pixel):
     * </p>
     * <table>
     * <thead>
     * <tr>
     * <th align="center"></th>
     * <th align="center">bit 7</th>
     * <th align="center">bit 6</th>
     * <th align="center">bit 5</th>
     * <th align="center">bit 4</th>
     * <th align="center">bit 3</th>
     * <th align="center">bit 2</th>
     * <th align="center">bit 1</th>
     * <th align="center">bit 0</th>
     * </tr>
     * </thead> <tbody>
     * <tr>
     * <td align="center">Byte 0:</td>
     * <td align="center">P0[9]</td>
     * <td align="center">P0[8]</td>
     * <td align="center">P0[7]</td>
     * <td align="center">P0[6]</td>
     * <td align="center">P0[5]</td>
     * <td align="center">P0[4]</td>
     * <td align="center">P0[3]</td>
     * <td align="center">P0[2]</td>
     * </tr>
     * <tr>
     * <td align="center">Byte 1:</td>
     * <td align="center">P1[9]</td>
     * <td align="center">P1[8]</td>
     * <td align="center">P1[7]</td>
     * <td align="center">P1[6]</td>
     * <td align="center">P1[5]</td>
     * <td align="center">P1[4]</td>
     * <td align="center">P1[3]</td>
     * <td align="center">P1[2]</td>
     * </tr>
     * <tr>
     * <td align="center">Byte 2:</td>
     * <td align="center">P2[9]</td>
     * <td align="center">P2[8]</td>
     * <td align="center">P2[7]</td>
     * <td align="center">P2[6]</td>
     * <td align="center">P2[5]</td>
     * <td align="center">P2[4]</td>
     * <td align="center">P2[3]</td>
     * <td align="center">P2[2]</td>
     * </tr>
     * <tr>
     * <td align="center">Byte 3:</td>
     * <td align="center">P3[9]</td>
     * <td align="center">P3[8]</td>
     * <td align="center">P3[7]</td>
     * <td align="center">P3[6]</td>
     * <td align="center">P3[5]</td>
     * <td align="center">P3[4]</td>
     * <td align="center">P3[3]</td>
     * <td align="center">P3[2]</td>
     * </tr>
     * <tr>
     * <td align="center">Byte 4:</td>
     * <td align="center">P3[1]</td>
     * <td align="center">P3[0]</td>
     * <td align="center">P2[1]</td>
     * <td align="center">P2[0]</td>
     * <td align="center">P1[1]</td>
     * <td align="center">P1[0]</td>
     * <td align="center">P0[1]</td>
     * <td align="center">P0[0]</td>
     * </tr>
     * </tbody>
     * </table>
     * <p>
     * This format assumes
     * <ul>
     * <li>a width multiple of 4 pixels</li>
     * <li>an even height</li>
     * </ul>
     * </p>
     *
     * <pre>size = row stride * height</pre> where the row stride is in <em>bytes</em>,
     * not pixels.
     *
     * <p>
     * Since this is a densely packed format, the pixel stride is always 0. The
     * application must use the pixel data layout defined in above table to
     * access each row data. When row stride is equal to {@code width * (10 / 8)}, there
     * will be no padding bytes at the end of each row, the entire image data is
     * densely packed. When stride is larger than {@code width * (10 / 8)}, padding
     * bytes will be present at the end of each row.
     * </p>
     * <p>
     * For example, the {@link android.media.Image} object can provide data in
     * this format from a {@link android.hardware.camera2.CameraDevice} (if
     * supported) through a {@link android.media.ImageReader} object. The
     * {@link android.media.Image#getPlanes() Image#getPlanes()} will return a
     * single plane containing the pixel data. The pixel stride is always 0 in
     * {@link android.media.Image.Plane#getPixelStride()}, and the
     * {@link android.media.Image.Plane#getRowStride()} describes the vertical
     * neighboring pixel distance (in bytes) between adjacent rows.
     * </p>
     *
     * @see android.media.Image
     * @see android.media.ImageReader
     * @see android.hardware.camera2.CameraDevice
     */
    public static final int RAW10 = 0x25;

    /**
     * <p>
     * Android 12-bit raw format
     * </p>
     * <p>
     * This is a single-plane, 12-bit per pixel, densely packed (in each row),
     * unprocessed format, usually representing raw Bayer-pattern images coming
     * from an image sensor.
     * </p>
     * <p>
     * In an image buffer with this format, starting from the first pixel of each
     * row, each two consecutive pixels are packed into 3 bytes (24 bits). The first
     * and second byte contains the top 8 bits of first and second pixel. The third
     * byte contains the 4 least significant bits of the two pixels, the exact layout
     * data for each two consecutive pixels is illustrated below (Pi[j] stands for
     * the jth bit of the ith pixel):
     * </p>
     * <table>
     * <thead>
     * <tr>
     * <th align="center"></th>
     * <th align="center">bit 7</th>
     * <th align="center">bit 6</th>
     * <th align="center">bit 5</th>
     * <th align="center">bit 4</th>
     * <th align="center">bit 3</th>
     * <th align="center">bit 2</th>
     * <th align="center">bit 1</th>
     * <th align="center">bit 0</th>
     * </tr>
     * </thead> <tbody>
     * <tr>
     * <td align="center">Byte 0:</td>
     * <td align="center">P0[11]</td>
     * <td align="center">P0[10]</td>
     * <td align="center">P0[ 9]</td>
     * <td align="center">P0[ 8]</td>
     * <td align="center">P0[ 7]</td>
     * <td align="center">P0[ 6]</td>
     * <td align="center">P0[ 5]</td>
     * <td align="center">P0[ 4]</td>
     * </tr>
     * <tr>
     * <td align="center">Byte 1:</td>
     * <td align="center">P1[11]</td>
     * <td align="center">P1[10]</td>
     * <td align="center">P1[ 9]</td>
     * <td align="center">P1[ 8]</td>
     * <td align="center">P1[ 7]</td>
     * <td align="center">P1[ 6]</td>
     * <td align="center">P1[ 5]</td>
     * <td align="center">P1[ 4]</td>
     * </tr>
     * <tr>
     * <td align="center">Byte 2:</td>
     * <td align="center">P1[ 3]</td>
     * <td align="center">P1[ 2]</td>
     * <td align="center">P1[ 1]</td>
     * <td align="center">P1[ 0]</td>
     * <td align="center">P0[ 3]</td>
     * <td align="center">P0[ 2]</td>
     * <td align="center">P0[ 1]</td>
     * <td align="center">P0[ 0]</td>
     * </tr>
     * </tbody>
     * </table>
     * <p>
     * This format assumes
     * <ul>
     * <li>a width multiple of 4 pixels</li>
     * <li>an even height</li>
     * </ul>
     * </p>
     *
     * <pre>size = row stride * height</pre> where the row stride is in <em>bytes</em>,
     * not pixels.
     *
     * <p>
     * Since this is a densely packed format, the pixel stride is always 0. The
     * application must use the pixel data layout defined in above table to
     * access each row data. When row stride is equal to {@code width * (12 / 8)}, there
     * will be no padding bytes at the end of each row, the entire image data is
     * densely packed. When stride is larger than {@code width * (12 / 8)}, padding
     * bytes will be present at the end of each row.
     * </p>
     * <p>
     * For example, the {@link android.media.Image} object can provide data in
     * this format from a {@link android.hardware.camera2.CameraDevice} (if
     * supported) through a {@link android.media.ImageReader} object. The
     * {@link android.media.Image#getPlanes() Image#getPlanes()} will return a
     * single plane containing the pixel data. The pixel stride is always 0 in
     * {@link android.media.Image.Plane#getPixelStride()}, and the
     * {@link android.media.Image.Plane#getRowStride()} describes the vertical
     * neighboring pixel distance (in bytes) between adjacent rows.
     * </p>
     *
     * @see android.media.Image
     * @see android.media.ImageReader
     * @see android.hardware.camera2.CameraDevice
     */
    public static final int RAW12 = 0x26;

    /**
     * <p>Android dense depth image format.</p>
     *
     * <p>Each pixel is 16 bits, representing a depth ranging measurement from a depth camera or
     * similar sensor. The 16-bit sample consists of a confidence value and the actual ranging
     * measurement.</p>
     *
     * <p>The confidence value is an estimate of correctness for this sample.  It is encoded in the
     * 3 most significant bits of the sample, with a value of 0 representing 100% confidence, a
     * value of 1 representing 0% confidence, a value of 2 representing 1/7, a value of 3
     * representing 2/7, and so on.</p>
     *
     * <p>As an example, the following sample extracts the range and confidence from the first pixel
     * of a DEPTH16-format {@link android.media.Image}, and converts the confidence to a
     * floating-point value between 0 and 1.f inclusive, with 1.f representing maximum confidence:
     *
     * <pre>
     *    ShortBuffer shortDepthBuffer = img.getPlanes()[0].getBuffer().asShortBuffer();
     *    short depthSample = shortDepthBuffer.get()
     *    short depthRange = (short) (depthSample & 0x1FFF);
     *    short depthConfidence = (short) ((depthSample >> 13) & 0x7);
     *    float depthPercentage = depthConfidence == 0 ? 1.f : (depthConfidence - 1) / 7.f;
     * </pre>
     * </p>
     *
     * <p>This format assumes
     * <ul>
     * <li>an even width</li>
     * <li>an even height</li>
     * <li>a horizontal stride multiple of 16 pixels</li>
     * </ul>
     * </p>
     *
     * <pre> y_size = stride * height </pre>
     *
     * When produced by a camera, the units for the range are millimeters.
     */
    public static final int DEPTH16 = 0x44363159;

    /**
     * Android sparse depth point cloud format.
     *
     * <p>A variable-length list of 3D points plus a confidence value, with each point represented
     * by four floats; first the X, Y, Z position coordinates, and then the confidence value.</p>
     *
     * <p>The number of points is {@code (size of the buffer in bytes) / 16}.
     *
     * <p>The coordinate system and units of the position values depend on the source of the point
     * cloud data. The confidence value is between 0.f and 1.f, inclusive, with 0 representing 0%
     * confidence and 1.f representing 100% confidence in the measured position values.</p>
     *
     * <p>As an example, the following code extracts the first depth point in a DEPTH_POINT_CLOUD
     * format {@link android.media.Image}:
     * <pre>
     *    FloatBuffer floatDepthBuffer = img.getPlanes()[0].getBuffer().asFloatBuffer();
     *    float x = floatDepthBuffer.get();
     *    float y = floatDepthBuffer.get();
     *    float z = floatDepthBuffer.get();
     *    float confidence = floatDepthBuffer.get();
     * </pre>
     *
     * For camera devices that support the
     * {@link android.hardware.camera2.CameraCharacteristics#REQUEST_AVAILABLE_CAPABILITIES_DEPTH_OUTPUT DEPTH_OUTPUT}
     * capability, DEPTH_POINT_CLOUD coordinates have units of meters, and the coordinate system is
     * defined by the camera's pose transforms:
     * {@link android.hardware.camera2.CameraCharacteristics#LENS_POSE_TRANSLATION} and
     * {@link android.hardware.camera2.CameraCharacteristics#LENS_POSE_ROTATION}. That means the origin is
     * the optical center of the camera device, and the positive Z axis points along the camera's optical axis,
     * toward the scene.
     */
    public static final int DEPTH_POINT_CLOUD = 0x101;

    /**
     * Unprocessed implementation-dependent raw
     * depth measurements, opaque with 16 bit
     * samples.
     *
     * @hide
     */
    public static final int RAW_DEPTH = 0x1002;

    /**
     * Android private opaque image format.
     * <p>
     * The choices of the actual format and pixel data layout are entirely up to
     * the device-specific and framework internal implementations, and may vary
     * depending on use cases even for the same device. The buffers of this
     * format can be produced by components like
     * {@link android.media.ImageWriter ImageWriter} , and interpreted correctly
     * by consumers like {@link android.hardware.camera2.CameraDevice
     * CameraDevice} based on the device/framework private information. However,
     * these buffers are not directly accessible to the application.
     * </p>
     * <p>
     * When an {@link android.media.Image Image} of this format is obtained from
     * an {@link android.media.ImageReader ImageReader} or
     * {@link android.media.ImageWriter ImageWriter}, the
     * {@link android.media.Image#getPlanes() getPlanes()} method will return an
     * empty {@link android.media.Image.Plane Plane} array.
     * </p>
     * <p>
     * If a buffer of this format is to be used as an OpenGL ES texture, the
     * framework will assume that sampling the texture will always return an
     * alpha value of 1.0 (i.e. the buffer contains only opaque pixel values).
     * </p>
     */
    public static final int PRIVATE = 0x22;

    /**
     * Compressed HEIC format.
     *
     * <p>This format defines the HEIC brand of High Efficiency Image File
     * Format as described in ISO/IEC 23008-12.</p>
     */
    public static final int HEIC = 0x48454946;

    /**
     * Use this function to retrieve the number of bits per pixel of an
     * ImageFormat.
     *
     * @param format
     * @return the number of bits per pixel of the given format or -1 if the
     *         format doesn't exist or is not supported.
     */
    public static int getBitsPerPixel(@Format int format) {
        switch (format) {
            case RGB_565:
                return 16;
            case NV16:
                return 16;
            case YUY2:
                return 16;
            case YV12:
                return 12;
            case Y8:
                return 8;
            case Y16:
            case DEPTH16:
                return 16;
            case NV21:
                return 12;
            case YUV_420_888:
                return 12;
            case YUV_422_888:
                return 16;
            case YUV_444_888:
                return 24;
            case FLEX_RGB_888:
                return 24;
            case FLEX_RGBA_8888:
                return 32;
            case RAW_DEPTH:
            case RAW_SENSOR:
                return 16;
            case RAW10:
                return 10;
            case RAW12:
                return 12;
        }
        return -1;
    }

    /**
     * Determine whether or not this is a public-visible {@code format}.
     *
     * <p>In particular, {@code @hide} formats will return {@code false}.</p>
     *
     * <p>Any other formats (including UNKNOWN) will return {@code false}.</p>
     *
     * @param format an integer format
     * @return a boolean
     *
     * @hide
     */
    public static boolean isPublicFormat(@Format int format) {
        switch (format) {
            case RGB_565:
            case NV16:
            case YUY2:
            case YV12:
            case JPEG:
            case NV21:
            case YUV_420_888:
            case YUV_422_888:
            case YUV_444_888:
            case FLEX_RGB_888:
            case FLEX_RGBA_8888:
            case RAW_SENSOR:
            case RAW_PRIVATE:
            case RAW10:
            case RAW12:
            case DEPTH16:
            case DEPTH_POINT_CLOUD:
            case PRIVATE:
            case RAW_DEPTH:
            case Y8:
            case DEPTH_JPEG:
            case HEIC:
                return true;
        }

        return false;
    }
}

```

