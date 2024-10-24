## 三种类型

 - 组合现有控件

把现有控件组合起来形成一个新的控件

- 继承现有控件

在该控件的基础之上添加新功能

- 直接继承 View / ViewGroup



## 知识点

- ViewGroup 也继承 VIew ，它可以包含子 View

- 代码中调用 View(Context context) 构造函数实例化 View，xml 中调用 View(Context context, AttributeSet attrs) 构造函数实例化 View

- View 树结构，从根节点开始

- 屏幕坐标系和数学坐标系不太一样 ，屏幕左上角是坐标原点，Y 轴往下增大

- 角度(angle)   360°  屏幕坐标系中角度增大方向为顺时针，而数学坐标系则是逆时针

- 弧度(radian)  弧长等于半径时对应的夹角就是1弧度，1rad ≈ **57.296**° ，1° ≈ **0.017**rad , 

  > 1rad = ?°  -> 圆周长 = 360 ->  2πr = 360  ->   (2π)rad  = 360° -> 1rad = (360/2π)° -> 1rad = (180/π)°
  >
  > 1°=?rad   -> 圆周长 = 360 ->  2πr = 360  ->  (2π)rad   = 360°   ->  1°= (2π/360)rad  -> 1°= (π/180)rad
  >
  > 当角度以弧度形式给出时，通常不写弧度单位，直接写值

- 颜色模式
   
   >  ARGB8888 、ARGB4444 、RGB565 (安卓默认模式) 、Alpha8 



## 自定义 View 的基本流程

- 1 重写构造方法，通常是 4 个构成方法，至少重写一个，初始化和获取自定义属性
- 2 重写 onMeasure ，测量 View 大小
- 3 重写 onSizeChanged ，确定 View 大小
- 4 重写 onLayout ，确定子 View 布局（ViewGroup 才有）
- 5 重写 onDraw，绘制内容



## 具体流程

>  measure、layout、draw 



### View 的 measure 测量流程

> measure -> onMeasure -> getDefaultSize -> setMeasuredDimension

## View.measure(int widthMeasureSpec, int heightMeasureSpec)

- 测量确认 View 的宽高

- 先测量子view的大小，最后测量自身的大小（ViewGroup）
- 可能需要多次测量，不一定准确，推荐在 onLayout 中得到最终的宽高尺寸

- 内部调用过 View.onMeasure

### View.onMeasure(int widthMeasureSpec, int heightMeasureSpec)

- 测量 View 大小

```java
// onMeasure 默认实现 一般需要复写
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

该方法最终目的就是调用 setMeasuredDimension 设置测量后的尺寸

```java
//计算View的尺寸
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    //得到测量的模式 UNSPECIFIED、EXACTLY、AT_MOST 
    int specMode = MeasureSpec.getMode(measureSpec);
    //得到具体的数值
    int specSize = MeasureSpec.getSize(measureSpec);
    switch (specMode) {
    //默认值，父控件没有给子View任何限制，子View可以设置为任意大小
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST://子View具体大小没有尺寸限制，但是存在上限，上限一般为父View大小
    case MeasureSpec.EXACTLY://父控件已经确切的指定了子View的大小
        result = specSize;
        break;
    }
    return result;
}
```

- MeasureSpec 测量规格，View.MeasureSpec 封装了 Mode 和 Size 

```java
EXACTLY  精确值和 match_parent
AT_MOST  wrap_content
UNSPECIFIED 无限制
//包装 Mode 和 Size  
//int 类型 32位 -> 2位存 mode，30位存 size
int measureSpec = MeasureSpec.makeMeasureSpec(int size, int mode)
MeasureSpec 原理涉及到以下二进制位运算
~ 位取反
& 按位与
| 按位或
<< 左移
```



# ViewGroup 的 measure 测量流程

> measure -> onMeasure -> measureChildren -> measureChild -> getChildMeasureSpec -> child.measure  遍历测量子 View -> setMeasuredDimension

ViewGroup.measureChildren(int widthMeasureSpec, int heightMeasureSpec)

```java
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}
```

ViewGroup.measureChild(View child, int parentWidthMeasureSpec,int parentHeightMeasureSpec)

```java
protected void measureChild(View child, int parentWidthMeasureSpec,
        int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();
    //子view的大小由父view的MeasureSpec值 和 子view的LayoutParams属性 共同决定
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```





### onSizeChanged(int w, int h, int oldw, int oldh)

- 确定 View 大小，既然上面已经测量过 View 的大小了，那为啥还要确定一次？因为 measure 测量是 View 自己的大小，而 View 能展现的大小还受父控件的影响



## View  layout 流程

- layout 确认 View 坐标位置

- 先确定自身的位置，然后调用child.layout()确定子类位置

  onLayout，setFrame



## View.layout(int l, int t, int r, int b)

- 可能会调用 View.onMeasure
- 内部调用过 View.onLayout

### View.onLayout(boolean changed, int left, int top, int right, int bottom)



### ViewGroup.layout(int l, int t, int r, int b)

- 内部调用过 super.layout (就是 View.layout)

  

### ViewGroup.onLayout(boolean changed,int l, int t, int r, int b)

- 确定子 View 的布局位置，** getXXX 都是相对于 parentView  父控件**

childView.getLeft  代表 childView 的左侧到 parentView 左侧的距离

childView.getTop  代表 childView 的上侧到 parentView 上侧的距离

childView.getRight  代表 childView 的右侧到 parentView 左侧的距离

childView.getBottom  代表 childView 的下侧到 parentView 上侧的距离





### requestLayout







## View  draw 流程

> draw -> drawBackground -> onDraw -> dispatchDraw -> onDrawForeground

- draw 绘制 View 到屏幕上

## onDraw(Canvas canvas)

> 

### Paint 




# Canvas 画布





 



## MotionEvent



#### 处理 View 的事件冲突

- 外部拦截法	
- 内部拦截法





## Scroller







## invalidate() 和 postInvalidate() 的区别