# 在需要时刻刷新某一区域的组件时，建议通过以下方式避免引发全局 layout刷新

在需要时刻刷新某一区域的组件时，建议通过以下方式避免引发全局 layout
刷新:

1)  设置固定的 view 大小的高宽，如倒计时组件等；
2)  调用 view 的 layout 方式修改位置，如弹幕组件等；

3)  通过修改 canvas 位置并且调用 invalidate(int l, int t, int r, int b)等方式限定刷新
区域；
4)  通过设置一个是否允许 requestLayout 的变量，然后重写控件的 requestlayout、
onSizeChanged 方法 ， 判 断 控 件 的大小 没 有 改 变 的 情况下 ， 当 进 入
requestLayout 的时候，直接返回而不调用 super 的 requestLayout 方法。