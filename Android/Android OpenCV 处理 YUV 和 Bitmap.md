# OpenCV 处理 YUV 和 Bitmap

- Mat 矩阵信息
- Mat 类可以分为两个部分：矩阵头和指向像素数据的矩阵指针
- 里面部分借鉴了 Matlab 的思路和风格

```kotlin
fun nv12ToBitmap(byteArray: ByteArray, size: Size): Bitmap {
        val rgbaMat = nv12ToRgbaMat(byteArray, size)
        //
        val bitmap = Bitmap.createBitmap(size.width, size.height, Bitmap.Config.ARGB_8888)
        Utils.matToBitmap(rgbaMat, bitmap)
        return bitmap
    }
```

```kotlin
    fun nv12ToRgbaMat(byteArray: ByteArray, size: Size): Mat {
        //rows 行,cols 列,type
        val mat = Mat(size.height, size.width, CvType.CV_8UC1) //U无符号整形 C通道
//        val mat = Mat()
//        mat.create(size.height, size.width,CvType.CV_8UC1)
//        mat.create(size,CvType.CV_8UC1)
        mat.put(0, 0, byteArray)

        val bytesXXX = ByteArray(size.width * size.height / 2)
        System.arraycopy(byteArray, size.width * size.height, bytesXXX, 0, bytesXXX.size)

        val mat2 = Mat(size.height / 2, size.width / 2, CvType.CV_8UC2)
        mat2.put(0, 0, bytesXXX)

        val rgbaMat = Mat()

        Imgproc.cvtColorTwoPlane(mat, mat2, rgbaMat, Imgproc.COLOR_YUV2RGBA_NV12)

        return rgbaMat
    }
```

```kotlin
    fun bitmapToYuvI420(bitmap: Bitmap): ByteArray {
        val rgbaMat = Mat(bitmap.height, bitmap.width, CvType.CV_8UC4)
        Utils.bitmapToMat(bitmap, rgbaMat)
        val bytesI420 = rgbaMatToYuvI420(rgbaMat)
        return bytesI420
    }

```


```kotlin
    fun rgbaMatToYuvI420(rgbaMat: Mat): ByteArray {
        val size = Size(rgbaMat.width(), rgbaMat.height())
        val bytesI420 = ByteArray(size.width * size.height * 3 / 2)

        val matI420 = Mat()

        Imgproc.cvtColor(rgbaMat, matI420, Imgproc.COLOR_RGBA2YUV_I420)

        matI420.get(0, 0, bytesI420)

        return bytesI420
    }
    
```