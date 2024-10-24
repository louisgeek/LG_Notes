
# Android 事件分发机制
- View#dispatchTouchEvent(MotionEvent event)
- ViewGroup#onInterceptTouchEvent(MotionEvent event)
- View#onTouchEvent(MotionEvent event)
- ViewGroup#requestDisallowInterceptTouchEvent(boolean disallowIntercept)


```java

/**
 * 伪代码
 * ViewGroup 的三个核心方法
 */
public boolean dispatchTouchEvent(MotionEvent event) {
    //dispatchTouchEvent 进行事件分发
    //onInterceptTouchEvent 判断是否拦截事件，只有 ViewGroup 有这个方法
    if (onInterceptTouchEvent(event)) {
        //onTouchEvent 处理事件
        return onTouchEvent(event);
    }
    return child.dispatchTouchEvent(event);
}
```

## 事件流
- 事件传递的顺序 Activity -> Window（PhoneWindow） -> DecorView（FrameLayout） -> ViewGroup -> View


### Activity 事件分发

```java
 Activity#dispatchTouchEvent
 -PhoneWindow#superDispatchTouchEvent
 --DecorView#superDispatchTouchEvent
 ---super.dispatchTouchEvent      //此处事件就传递到 ViewGroup 里去了，实现了事件开始从 Activity 然后直到 ViewGroup 里
 -Activity#onTouchEvent           //如果事件未被上述 Activity 内的任何视图消费就会走这个
 --Window#shouldCloseOnTouch      //Dialog 类的 Activity 点击对话框外侧后弹框消失
 --Activity#finish                //如果 shouldCloseOnTouch 返回 true 就会走这个
```
### ViewGroup 事件分发

```java
ViewGroup#dispatchTouchEvent
-判断 disallowIntercept 或者 ViewGroup#onInterceptTouchEvent 的返回值取反  //ViewGroup#onInterceptTouchEvent 默认返回 false，可按需重写
-如果满足上述条件则继续通过 for 循环遍历当前 ViewGroup 下的所有子 View
--View#dispatchTouchEvent                                                 //如果内部消费了就返回 false，然后针对 ViewGroup#dispatchTouchEvent 就直接被返回 true 了
-super#dispatchTouchEvent                                                 //如果事件未满足上述条件或未被任何子 View 消费就会走这个，此处事件就传递到 View 里去了
--View#onTouchEvent
---View#performClick
----View#onClick
```

## ViewGroup#requestDisallowInterceptTouchEvent(boolean disallowIntercept)
- 请求不拦截，disallowIntercept 默认 false


## 实例分析
### 点击一个按钮
- ViewGroup 默认不拦截
- 满足条件走 for 循环遍历子 View，找到这个按钮
- 然后走 View#dispatchTouchEvent -> View#onTouchEvent -> View#performClick -> View#onClick，执行了 View 的点击事件
- 此时 ViewGroup#dispatchTouchEvent 就直接被返回 true 了，所以 ViewGroup 的点击事件是不会走的

### 点击空白区域
- ViewGroup 默认不拦截
- 满足条件走 for 循环遍历子 View，未找到子 View
- 然后 ViewGroup 自己执行 super#dispatchTouchEvent 
- 也就是此时走 View#dispatchTouchEvent -> View#onTouchEvent -> View#performClick -> View#onClick，也就是执行了 ViewGroup 自己的点击事件