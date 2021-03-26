1、抽象类

```java
//实现 View.OnClickListener 接口
public abstract class OnThrottleClickListener implements View.OnClickListener {
    private static final String TAG = "OnThrottleClickListener";
    private static final int SKIP_DURATION = 300;//milliseconds
    private long mLastClickTime;
    @Override
    public void onClick(View v) {
        if (System.currentTimeMillis() - mLastClickTime > SKIP_DURATION) {
            onThrottleClick(v);
            mLastClickTime = System.currentTimeMillis();
        } else {
            Log.e(TAG, "OnThrottleClickListener: 重复点击");
        }
    }

    protected abstract void onThrottleClick(View v);
}
```

```java
//代替 new View.OnClickListener() 使用
id_tv_1.setOnClickListener(new OnThrottleClickListener() {
     @Override
     protected void onThrottleClick(View v) {
       Log.e(TAG, "onClick: OnThrottleClickListener ");
     }
});
```



2、代理模式

```java
//代理类实现 View.OnClickListener 接口
public class ThrottleClickProxy implements View.OnClickListener {
    private static final String TAG = "ThrottleClickProxy";
    private static final int SKIP_DURATION = 300;//milliseconds
    private long mLastClickTime;
    //源对象
    private View.OnClickListener mOriginListener;
	//构造
    public ThrottleClickProxy(View.OnClickListener mOriginListener) {
        this.mOriginListener = mOriginListener;
    }
    @Override
    public void onClick(View v) {
        if (System.currentTimeMillis() - mLastClickTime >= SKIP_DURATION) {
            mOriginListener.onClick(v);
            mLastClickTime = System.currentTimeMillis();
        } else {
            Log.e(TAG, "ThrottleClickProxy: 重复点击");
        }
    }
}
```

```java
//使用 
id_tv_2.setOnClickListener(new ThrottleClickProxy(new View.OnClickListener() {
          @Override
          public void onClick(View v) {
                Log.e(TAG, "onClick: ThrottleClickProxy ");
          }
   }));
```



3、RxAndroid 之 RxBinding

```groovy
implementation 'com.jakewharton.rxbinding3:rxbinding:3.0.0-alpha1'
```

```java
 RxView.clicks(id_tv_3)
       .throttleFirst(300, TimeUnit.MILLISECONDS)
       .subscribe(new Consumer<Unit>() {
          @Override
          public void accept(Unit unit) throws Exception {
              Log.e(TAG, "onClick: throttleFirst ");
          }
     });
```



4、AOP 之 Eclipse AspectJ

```groovy
//采用 AspectJX 来快速配置 Eclipse AspectJ
//project gradle
dependencies {
        classpath "com.android.tools.build:gradle:4.1.2"
        //add
        classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.10'
}
```

```groovy
plugins {
    id 'com.android.application'
    //add
    id 'android-aspectjx'
}
//module gradle
dependencies {
    //add
    implementation 'org.aspectj:aspectjrt:1.9.5'
}
```

```java
@Aspect
public class ThrottleClickAspect {
    private static final String TAG = "ThrottleClickAspect";
    private static final int SKIP_DURATION = 3000;
    private long mLastClickTime;

    //所有的 android.view.View.OnClickListener.onClick 方法都会被织入，像第三方组件 RxView.clicks() 里也会
    @Around("execution(* android.view.View.OnClickListener.onClick(..))")
    public void aroundViewOnClick(ProceedingJoinPoint joinPoint) throws Throwable {
        if (System.currentTimeMillis() - mLastClickTime >= SKIP_DURATION) {
            joinPoint.proceed();
            mLastClickTime = System.currentTimeMillis();
        } else {
            Log.e(TAG, "ThrottleClickAspect: 重复点击");
        }
    }
}
```

- 代码无侵入方式，直接生效了

通过注解明确指定

- 增加 @ThrottleClick 注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ThrottleClick {
    int skipDuration() default 1000;
}
```
- 修改 ThrottleClickAspect 类

```java
@Aspect
public class ThrottleClickAspect {
    private static final String TAG = "ThrottleClickAspect";
    private static final int SKIP_DURATION = 300;
    //    private int mLastClickViewId;
    private long mLastClickTime;
    //声明是注解标记切点
	//包路径
    @Pointcut("execution(@com.louisgeek.aop.ThrottleClick * *(..))")
    public void pointcutThrottleClick() {
        Log.e(TAG, "pointcutThrottleClick: ");
    }
	//
    @Around("pointcutThrottleClick()")
    public void aroundThrottleClick(ProceedingJoinPoint joinPoint) throws Throwable {
        //
        View view = null;
        for (Object arg : joinPoint.getArgs()) {
            if (arg instanceof View) {
                view = (View) arg;
                break;
            }
        }
        if (view == null) {
            return;
        }
        // 取出方法的注解
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        Method method = methodSignature.getMethod();
        if (!method.isAnnotationPresent(ThrottleClick.class)) {
            return;
        }
        ThrottleClick throttleClick = method.getAnnotation(ThrottleClick.class);
        //
        if (throttleClick == null) {
            return;
        }
        int skipDuration = throttleClick.skipDuration();
        if (skipDuration < 0) {
            skipDuration = SKIP_DURATION;
        }
        if (System.currentTimeMillis() - mLastClickTime >= skipDuration) {
            //继续执行
            joinPoint.proceed();
            mLastClickTime = System.currentTimeMillis();
        } else {
            Log.e(TAG, "aroundThrottleClick: 重复点击");
        }
    }

}
```

```java
//使用  
id_tv_4.setOnClickListener(new View.OnClickListener() {
           //注解标记
            @ThrottleClick
            @Override
            public void onClick(View v) {
                Log.e(TAG, "onClick: @ThrottleClick ");
            }
  });
```

