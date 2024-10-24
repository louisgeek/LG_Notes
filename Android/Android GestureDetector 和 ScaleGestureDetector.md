# Android GestureDetector 和 ScaleGestureDetector

## GestureDetector 手势检测
- 可接管 View 的 onTouchEvent 方法
```
 override fun onTouchEvent(event: MotionEvent?): Boolean {
//        return super.onTouchEvent(event)
        return gestureDetector.onTouchEvent(event)
                ||super.onTouchEvent(event)
    }

- GestureDetector.OnGestureListener 接口，包括 onScroll 滑动、onFling 抛、onLongPress 长按、onSingleTapUp 点按抬起等
- GestureDetector.OnDoubleTapListener 接口，包括 onSingleTapConfirmed 单击、onDoubleTap 双击等
- GestureDetector.OnContextClickListener 接口，外接键盘、蓝牙外设等事件


```
- GestureDetector.SimpleOnGestureListener 静态内部类，实现了 OnGestureListener, OnDoubleTapListener, OnContextClickListener，方法都是默认空实现




## ScaleGestureDetector
- 可以接管 View 的 onTouchEvent 方法
```
   override fun onTouchEvent(event: MotionEvent?): Boolean {
//        return super.onTouchvent(event)
        return scaleGestureDetector.onTouchEvent(event)
                ||super.onTouchEvent(event)
    }
```
- ScaleGestureDetector.OnScaleGestureListener 接口，包括 onScale、onScaleBegin、onScaleEnd 方法
- ScaleGestureDetector.SimpleOnScaleGestureListener 静态内部类，实现了 OnScaleGestureListener，方法都是默认空实现





## GestureDetector 和 ScaleGestureDetector 组合使用

```
       override fun onTouchEvent(event: MotionEvent): Boolean {
   //        return super.onTouchEvent(event)
           if (event.pointerCount >= 2) {
   //            return scaleGestureDetector.onTouchEvent(event)|| super.onTouchEvent(event)
               return scaleGestureDetector.onTouchEvent(event)
           }
   //        return gestureDetector.onTouchEvent(event)|| super.onTouchEvent(event)
           return gestureDetector.onTouchEvent(event)
       }
```