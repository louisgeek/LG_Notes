

## FaceDetector
- 从相机中获取到数据转成 bitmap 再进行人脸识别

```kotlin
 val maxFaces = 3
        val faceDetector = FaceDetector(bitmap.width, bitmap.height, maxFaces)
        val faces: Array<FaceDetector.Face?> = arrayOfNulls(maxFaces)
        val faceCount = faceDetector.findFaces(bitmap, faces)
        if (faceCount > 0) {
            faces[0]?.let { face ->
                //人脸的可性度
                val confidence = face.confidence()
                //两眼的距离
                val eyesDistance = face.eyesDistance()
            }
        }
```
 
 

 