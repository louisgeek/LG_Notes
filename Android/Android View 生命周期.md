 
 ## 回顾 Activity 生命周期
- 6个
```
 onCreate —— onStart 可见 —— onResume 有焦点 —— onPause 无焦点 —— onStop 不可见 —— onDestory
```

## View 生命周期

```
//默认 visibility = VISIBLE
 onFinishInflate —— onAttachedToWindow —— onWindowVisibilityChanged(VISIBLE) —— onMeasure —— onSizeChanged —— onLayout —— onDraw —— onWindowFocusChanged(true)  —— onWindowVisibilityChanged(GONE) —— onWindowFocusChanged(false) —— onDetachedFromWindow
```
 - onFinishInflate 在 Activity 的 onCreate 方法调用后执行，当 View 及其子 View 从 xml 文件中加载完成后调用
 - onAttachedToWindow 在 Activity 的 onResume 方法调用后执行，Window 就是指 PhoneWindow
 - onWindowVisibilityChanged(VISIBLE) 紧跟着 onAttachedToWindow，不过锁屏的时候 onWindowVisibilityChanged(GONE) 不执行
 - onMeasure、onSizeChanged、onLayout、onDraw 一般在 onWindowVisibilityChanged(VISIBLE) 方法调用后执行，所以在 Activity 的 onResume 方法里是无法获取 View 正确的宽高的,前三个会根据实际情况调用多次
- onWindowFocusChanged 官方推荐采用 onWindowFocusChanged(true) 回调来确定当前 View 所在的 Activity 对用户可见且可交互的
- 还有一个额外的 onVisibilityChanged ，感觉不常用，暂时没有考虑进去
- 默认 visibility = GONE 就是不继续走 onMeasure、onSizeChanged、onLayout、onDraw
- 默认 visibility = INVISIBLE 就是不继续走 onDraw


## 生命周期使用场景

### onWindowFocusChanged
- 结合上文可以考虑在 onWindowFocusChanged(true) 里去获取当前 View 的宽高尺寸
- 可以考虑在 onWindowFocusChanged 开始或停止动画


 
### onDetachedFromWindow
- onDetachedFromWindow 很适合用来执行销毁自定义 View 使用的资源和动画等逻辑
 
 